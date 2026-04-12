# ERTP Alert Type Samples

Real examples from production surge alert files. Use these as templates when writing new alerts.

---

## 1. Muttley Error Rate

**When to use:** Alert on endpoint failure rate (`muttley.edge.failures / muttley.edge.calls`). The standard alert for RPC reliability on a gRPC procedure.

**Thresholds:** Fraction (e.g. `0.06` = 6%). Use `sustainPeriod: 300` for fast-moving, `1200` for sustained.

```yaml
- template: *templateZonal
  alert:
    name: Get Delivery Levers error rate is high
    description: There are high error rates in the get delivery levers endpoint. Check rt-surge-delivery ELK and dashboard to determine a root cause.
    sustainPeriod: 300
    warn: 0.06
    critical: 0.1
    query: |-
      fetch stat:muttley.edge.failures dest:rt-surge-delivery
        dest-zone:!routing-override
        procedure:rtsurgedelivery/getdeliverylevers
        error:!client-connection-failed
        | sum
        | asPercent (
          fetch stat:muttley.edge.calls dest:rt-surge-delivery
          dest-zone:!routing-override
          procedure:rtsurgedelivery/getdeliverylevers
          | sum
        )
        | removeEmpty
        | transformNull 0
        | moving 1m average
    actions: *allActions
```

---

## 2. Ratio / Percentage Alert

**When to use:** Alert when a metric's share of total exceeds a threshold — missing headers, error rate vs total calls, fallback rate, etc. Numerator / denominator both fetched explicitly.

**Thresholds:** Percentage points (e.g. `10` = 10%).

```yaml
- template: *template
  alert:
    name: Optionality required headers high missing rate
    description: "Fires if the % of calls missing citrus XP headers exceeds threshold."
    query: |-
      missing = fetch type:counter name:rollup.rt-surge-delivery.middleware.header-validator.header.missing
        header_name:{uberctx-logctxfx.appvariant,x-uber-app-variant}
        | sumSeries header_name | transformNull 0 | scaleToSeconds 1 | minSeries;

      total = fetch stat:muttley.edge.calls dest:rt-surge-delivery
        procedure:rtsurgedelivery/getdeliverylevers
        | sumSeries | transformNull 0 | scaleToSeconds 1;

      missing | asPercent(total) | moving 5m avg
    sustainPeriod: 600
    warn: 10
    critical: 15
    actions:
      pagerDutyActions: *pagerDutyAction
```

---

## 3. Raw Counter Spike

**When to use:** Alert when an error/anomaly counter that should be near zero starts firing — demand model missing, config fetch failures, invalid data. Simple `sumSeries` + moving window.

**Thresholds:** Absolute counts per window. `sustainPeriod: 300` typical.

```yaml
- template: *template
  alert:
    name: Lever Calculator - Demand Model Missing
    description: "A demand model was not found for some requests. Most likely causes are a newly launched country or misconfigured flipr."
    query: |-
      fetch name:baf_additive_demand_model_missing
        | sumSeries country_iso2
        | moving 2m sum
        | aliasByTags country_iso2
    warn: 0.5
    critical: 2
    sustainPeriod: 300
    actions:
      pagerDutyActions: *pagerDutyAction
```

---

## 4. Drop Below Threshold ("Low" Alert)

**When to use:** Alert when a metric that should always be non-zero drops — RR percentage, bytes read, task runs, presence of a value. Thresholds are near-zero; `warn` and `critical` are effectively the same (or `0` for critical).

**Thresholds:** Very small numbers (e.g. `0.000001`) signal "direction is downward." `sustainPeriod` can be short (60s) since drops are often sudden.

```yaml
- template: *template
  alert:
    name: Radius Reduction is low
    description: |-
      This alert fires when there has been a significant drop in RR across many markets.
    query: |-
      fetch service:storeindex name:storeindex.radius.reduction.store.count
        planname:storefrontfeedplanv2 env:production type:gauge
        | averageSeries uber_region
        | divideSeries (
          fetch service:storeindex name:storeindex.final.store.count
          planname:storefrontfeedplanv2 env:production type:gauge
          | averageSeries uber_region
        )
        | moving 5m avg | alias uber_region
    warn: .000001
    critical: 0
    sustainPeriod: 60
    actions:
      <<: *allActions
      pagerDutyActions: *pagerDutyAction
```

---

## 5. Week-over-Week (WoW)

**When to use:** Alert when today's metric is significantly lower (or higher) than the same time last week. Good for task runs, traffic volumes, or any metric with strong weekly seasonality. Uses `timeshift 7d` + `asPercent` + `offset -100`.

**Thresholds:** Percentage drop (e.g. `5` = 5% drop from last week). `sustainPeriod` can be short since WoW changes are gradual.

```yaml
- template:
    $ref: file:///umonitor-alert-group/templates/delivery/adhoc-template.yaml
  context:
    service: rt-surge-compute-delivery
  alert:
    name: Large drop in WoW task triggers
    description: |-
      There was too large of a drop in task triggers in the last week.
      Verify that there was a scale down and that this change is expected.
    query: |-
      TASK_CALLS = fetch service:rt-surge-compute-delivery* flow_id:primary
        name:calls
        task_type:{compute_delivery_surge_v2,save_delivery_matching_graph,compute_delivery_surge_fallback}
        | sumSeries task_type
        | moving 15m sum
        | transformNull 0;

      TASK_CALLS
        | asPercent (TASK_CALLS | timeshift 7d)
        | offset -100
        | scale -1
        | > 0
        | removeEmpty
    warn: 5
    critical: 8
    sustainPeriod: 60
```

---

## 6. Day-over-Day (DoD) — Presence Check

**When to use:** Alert when something was running yesterday (or the day before) but isn't running now. Used for scheduled compute tasks (Laminator runs, save tasks). The `divideSeries` produces NaN when the denominator is absent, so times with no historical baseline don't alert.

**Thresholds:** Near-zero (e.g. `0.0001`) — any value > 0 means today's run happened; near zero means it's missing.

```yaml
- template:
    $ref: file:///umonitor-alert-group/templates/delivery/adhoc-template.yaml
  context:
    service: rt-surge-compute-delivery
  alert:
    name: Missing save task for city (DoD)
    description: Save tasks were present yesterday or the day before at this time, but are not present now.
    query: |-
      D0 = fetch service:rt-surge-compute-delivery name:rollup.laminator_task_counts.*
        task_type:{save_delivery_matching_graph} flow_id:primary
        | averageSeries task_type region_id
        | moving 5m avg;
      D_MINUS_1 = D0 | timeshift -1d;
      D_MINUS_2 = D0 | timeshift -2d;
      NUM = D0 | transformNull 0;
      DEN = D_MINUS_1 | exec(D_MINUS_2) | max task_type region_id;
      DoD = NUM | divideSeries (DEN);
      DoD | removeEmpty | alias {{.task_type}}-city-{{.region_id}}
    warn: 0.0001
    critical: 0.000001
    sustainPeriod: 300
```

---

## 7. Histogram Latency (p99)

**When to use:** Alert on tail latency (p99, p95) for a service or datastore using histogram metrics. Requires `sumSeries bucketid bucket` + `histogramPercentile`. Values are in milliseconds.

**Reading histograms:** Histograms are emitted as bucketed counters. You **must** use `| histogramPercentile bucketid bucket <percentile>` to read them. Plain `sumSeries`, `.p99` suffixes, or `.upper` will **not** work.

**`moving avg` vs `moving sum`:** For latency alerts, use `moving Nm avg` — this gives the smoothed percentile value in the original unit (ms). Use `moving Nm sum` only when aggregating raw counts before computing a percentile.

**Thresholds:** Milliseconds (e.g. `20` = 20ms p99).

```yaml
- template: *template
  alert:
    name: High p99 latency for surge levers datastore
    description: "Unexpected sustained increase in p99 latency for surge levers datastore."
    query: |-
      fetch service:rt-surge-delivery
        name:rollup.surge_lever_serving_latency
        | sumSeries bucketid bucket service
        | moving 1m sum
        | histogramPercentile bucketid bucket 99
    warn: 20
    critical: 30
    sustainPeriod: 600
    actions:
      pagerDutyActions: *pagerDutyAction
```

---

## 8. Presence / One-off Event Alert

**When to use:** Alert when a single event that should never (or rarely) happen occurs — warmup failure, invalid proto, instance crash. `sustainPeriod: 0` fires immediately on first occurrence.

**Thresholds:** `warn: 1`, `critical: 2` (count of occurrences).

```yaml
- template: *template
  alert:
    name: Instance Warmup Failed
    description: When a new instance was spun up, warmup failed unexpectedly. If this happened during rollout, rollback and check for errors.
    query: |-
      fetch service:rt-surge-delivery name:warm_failed runtime_env:production
        | removeEmpty
        | sumSeries uber_zone
        | moving 15m sum
        | aliasByTags uber_zone
    warn: 1
    critical: 2
    sustainPeriod: 0
    actions: *allActions
```

---

## 9. Zonal vs LEGACY — When and How to Use Each

### What `type: ZONE` vs `type: LEGACY` means

- **ZONE**: The alert platform evaluates the query **per availability zone** (data center zone, e.g. PHX, DCA). If one zone breaches the threshold but others are healthy, only that zone fires.
- **LEGACY**: The alert evaluates **globally** across all zones. All zone data is aggregated into a single value.

In practice, **~80% of ERTP alerts use LEGACY and ~20% use ZONE**. LEGACY is the default choice; ZONE is used selectively for infrastructure and system health issues.

### When to use ZONE

Use ZONE when a single zone degrading independently is actionable and the issue is infrastructure or system-health related:
- **Muttley error rates** for core endpoints (e.g. getDeliveryLevers error rate)
- **Latency regressions** that vary by DC (step latency, p99 latency)
- **Instance/JVM health** (GC activity, warmup failures)
- **Cache latency** (Redis read/write latency per zone)
- **Compute task error rates** (triggers erroring out, chronos invocation failures)
- **External service integration failures** (circuit breakers, omniview fetch errors)
- **Data validation errors** (invalid proto data, missing pricer version)
- **Dampening anomalies** (dampening target read/write errors)

### When to use LEGACY

Use LEGACY for everything else — which is most alerts. This includes:
- **Pricing calculation issues** (demand model missing, DZR phi errors, invalid config, market failure rates)
- **Data quality and business logic** (radius reduction drops, missing base fees, low RR)
- **Feature/experiment monitoring** (missing headers, missing wiggles, ETD optionality variant, lever exemptions)
- **WoW and DoD comparisons** (traffic drops, missing tasks)
- **City/market-level performance** (task failures per city, merchant counts)
- **Redis operational metrics** (connection timeouts, cluster QPS spikes, low ops)
- **Rate limiting** (parameter serving rate limits, engagement gateway limits)
- **Demand modeling** (dampening target fetching, low DZR serving)
- **Storage and data growth** (storage growth, low bytes read)
- **Global ratio alerts** (numerator/denominator from different metric sources)

### Query differences: ZONE vs LEGACY

For ZONE alerts, end the query with `| sum` (no groupBy tags for zone — the platform handles zone splitting):

```yaml
# ZONE — platform splits by zone automatically
- template: *templateZonal
  alert:
    name: Get Delivery Levers error rate is high
    query: |-
      fetch stat:muttley.edge.failures dest:rt-surge-delivery
        dest-zone:!routing-override
        procedure:rtsurgedelivery/getdeliverylevers
        error:!client-connection-failed
        | sum
        | asPercent (
          fetch stat:muttley.edge.calls dest:rt-surge-delivery
          dest-zone:!routing-override
          procedure:rtsurgedelivery/getdeliverylevers
          | sum
        )
        | removeEmpty
        | transformNull 0
        | moving 1m average
```

For LEGACY alerts querying the same metric, aggregate explicitly by a dimension like `uber_region`:

```yaml
# LEGACY — aggregate across zones yourself
- template: *template
  alert:
    name: Get Delivery Levers error rate is high (service level)
    query: |-
      fetch stat:muttley.edge.failures dest:rt-surge-delivery
        dest-zone:!routing-override
        procedure:rtsurgedelivery/getdeliverylevers
        error:!client-connection-failed
        | sum uber_region
        | asPercent (
          fetch stat:muttley.edge.calls dest:rt-surge-delivery
          dest-zone:!routing-override
          procedure:rtsurgedelivery/getdeliverylevers
          | sum uber_region
        )
        | removeEmpty
        | transformNull 0
        | moving 1m average
```

### Template selection by service

| Service | ZONE template | LEGACY template |
|---------|--------------|-----------------|
| `rt-surge-delivery` | `*templateZonal` (inline anchor) | `*template` (inline anchor) |
| `rt-surge-compute-delivery` | `$ref: file:///umonitor-alert-group/templates/delivery/adhoc-zonal-template.yaml` | `$ref: file:///umonitor-alert-group/templates/delivery/adhoc-template.yaml` |
| `surge-data-delivery` | `$ref: file:///umonitor-alert-group/templates/delivery/adhoc-zonal.yaml` | `$ref: file:///umonitor-alert-group/templates/delivery/adhoc.yaml` |

### Threshold considerations for ZONE alerts

ZONE alerts evaluate per-zone, so thresholds should account for per-zone traffic (which is lower than global). This means:
- **Error rate thresholds can be the same** as LEGACY (they're percentages, zone-independent)
- **Absolute count thresholds should be lower** than LEGACY equivalents since each zone sees a fraction of traffic
- **`sustainPeriod` may need to be longer** for low-volume zones to avoid noise

### Pairing zonal + service-level alerts

A common pattern is to have **both** a ZONE and LEGACY alert for the same metric at different thresholds:

```yaml
# Tighter threshold, per-zone — catches single-zone degradation
- template: *templateZonal
  alert:
    name: Get Delivery Levers error rate is high - higher error rate
    warn: 0.06
    critical: 0.1
    sustainPeriod: 300

# Tighter threshold, global — catches widespread but lower-level degradation
- template: *template
  alert:
    name: Get Delivery Levers error rate is high - higher error rate (service level)
    warn: 0.04
    critical: 0.06
    sustainPeriod: 300
```

The service-level alert has **lower thresholds** because a 4% error rate across all zones is worse than 6% in one zone.

---

## Quick Reference

| Alert type | M3QL pattern | Threshold unit | `sustainPeriod` |
|-----------|-------------|---------------|-----------------|
| Muttley error rate | `failures \| asPercent(calls)` | Fraction (0.06 = 6%) | 300–1200s |
| Ratio / percentage | `numerator \| asPercent(denominator)` | Percentage points | 300–600s |
| Raw counter spike | `sumSeries \| moving Nm sum` | Absolute count | 180–300s |
| Drop below threshold | `divideSeries` or raw gauge | Near-zero (0.000001) | 60–300s |
| Week-over-Week | `asPercent(timeshift 7d) \| offset -100 \| scale -1` | % drop | 60s |
| Day-over-Day | `divideSeries(D_MINUS_1 \| exec(D_MINUS_2))` | Near-zero | 300s |
| Histogram latency | `histogramPercentile bucketid bucket 99` | Milliseconds | 300–600s |
| Presence / one-off | `moving 15m sum` | Count (1, 2) | 0 |

### Zonal vs LEGACY Quick Reference

| Factor | ZONE (`*templateZonal`) | LEGACY (`*template`) |
|--------|------------------------|---------------------|
| Evaluation | Per availability zone | Global across all zones |
| Usage share | ~20% of ERTP alerts | ~80% of ERTP alerts |
| When to use | Infrastructure/system health where one zone degrading is independently actionable | Business logic, data quality, pricing, features, global metrics |
| Query aggregation | `\| sum` (platform splits by zone) | `\| sum uber_region` or `\| sumSeries` (you aggregate) |
| Error rate thresholds | Can be looser (per-zone isolation) | Can be tighter (aggregated, so smaller deviations are meaningful) |
| Count thresholds | Lower (each zone sees fraction of traffic) | Higher (full traffic volume) |
| Default choice | No — use only for infra/system health | **Yes — default for most alerts** |
