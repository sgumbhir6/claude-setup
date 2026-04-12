---
name: understand-ertp-metrics
description: Use when needing to find a metric in the ERTP codebase, understand what it measures, confirm it exists in M3, or definitively conclude it does not exist in code
---

# ERTP: Understand Metric from Code

## Overview

Find the metric in code, trace what triggers it, confirm its full M3 name, and understand its tags. If exhaustive search finds nothing, say so confidently.

## Step 1: Search by concept, not metric name

You rarely know the exact metric string. Search by the concept — feature name, class name, or behavior:

```bash
cd ~/Uber/fievel
grep -r "citrus\|etd_surge\|missing_header" marketplace/surge/ --include="*.java" -l

# Then find M3 emit calls in those files
grep -n "counter\|gauge\|timer\|increment\|record" <file.java>
```

**Never start with an exact metric name string** — it rarely appears verbatim in code.

Also check constants files — but note that many metric names are **plain string literals** in the emit call, not defined in a constants file. Finding nothing in constants is expected; it doesn't mean the metric doesn't exist.
```bash
grep -r "MetricName\|METRIC_" marketplace/surge/ --include="*.java" -l
```

## Step 2: Trace the full M3 metric name

Java emits metrics via subScope chains. The full M3 name = service prefix + all subScope segments + metric name.

```java
// In ETDSurgeUtils:
metrics.subScope("etd_retriever").counter("xp_header_missing")
// → M3 name: rt-surge-delivery.etd_retriever.xp_header_missing
```

Trace the subScope chain from the root `M3Metrics` bean to the emit call. The service prefix (`rt-surge-delivery`, `rt-surge-compute-delivery`, `surge-data-delivery`) is added automatically by JFX.

**Watch for dynamically constructed names** — some metrics are built from runtime values:

```java
taggedMetrics.count("missing_" + headerName + "_" + callPath);
// → rt-surge-delivery.etd_retriever.missing_x-uber-client-version_some_call_path
```

These are high-cardinality and cannot be verified by name in M3. Document the pattern and what variables compose it — don't try to alert on them directly.

## Step 3: Read the emit site context

Once you find the emit call, read the surrounding method:
- What code path leads here? (request handler, Kafka consumer, scheduled job, error path)
- Happy path or error/fallback?
- Every request, or conditional? (feature flag, header presence, call path type)

## Step 4: Find the tags

Look for a `getMetricTags()` or tag-building method near the emit site.

Standard ERTP tags from `BaseContext.getMetricTags()` — these are always present, do not re-derive each time:

| Tag | Values | Notes |
|-----|--------|-------|
| `flow_id` | `primary`, shadow flow names | **Always filter `flow_id:primary`** in alerts |
| `runtime_env` | `production`, `shadow`, `staging` | Use for rollup metric filtering |
| `region_id` | integer city/region ID | Break down by city |
| `dc` | datacenter ID (e.g. `dca51`, `phx52`) | Available on rollups for zonal breakdowns |
| `tenancy` | `production`, `testing` | Rarely needed in alerts |
| `environment_type` | `production`, `shadow`, `staging` | Available on raw metrics but **not on rollups** — prefer `runtime_env` |
| `caller` | caller service name | Rarely needed |

**For alert query filtering, use `flow_id:primary`** — this is sufficient. Do not add `environment_type` as a filter.

## Step 5: Verify the metric exists in M3

Confirm it's actually emitting in production:

```
fetch service:rt-surge-delivery name:<full.metric.name> flow_id:primary
  | sumSeries | summarize 1h sum
```

If nothing appears, possible reasons:
- Metric is behind a feature flag
- Code path not yet deployed
- Metric name is wrong (check subScope chain again)
- Metric fires at near-zero rate — valid conclusion if the code path is rarely hit in production

## Definitively concluding "metric does not exist"

You can say **with confidence** the metric is not in the codebase only after all of:

- [ ] Searched for the concept keyword across `~/Uber/fievel/marketplace/surge/`
- [ ] Searched all constants/MetricName files for related strings
- [ ] Searched for the camelCase variant of any candidate name
- [ ] Checked M3 directly — metric does not appear in production data

If all four fail: **the metric does not exist**. Say so and recommend what existing metric to use or what to create.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Searching exact metric name string | Search by concept/feature keyword instead |
| Assuming subScope name = full metric name | Trace the full chain: service prefix + all subScopes + name |
| Stopping after one grep miss | Exhaust all 4 criteria before declaring "not found" |
| Skipping M3 verification | Always confirm metric exists in production |
| Treating "no data in M3" as definitive | Could be near-zero rate — check if code path is rarely hit |
| Searching constants file for all metrics | Many metrics are plain string literals; missing from constants is expected |
| Trying to alert on dynamic metric names | Document the pattern instead; use the stable companion counter for alerting |
