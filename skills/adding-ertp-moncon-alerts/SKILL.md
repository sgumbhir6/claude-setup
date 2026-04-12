---
name: adding-ertp-moncon-alerts
description: Use when adding a new Delivery surge alert in moncon — covers which file to edit, which template to reference, and the correct YAML structure per service. Output is the moncon alert YAML file edited with the new alert appended. See samples.md for real alert type examples.
---

# ERTP: Alert Structure and Templates

## Overview

ERTP alerts live in moncon YAML files under each service's `src/main/resources/moncon/umonitor-alert-group/` directory. Each Delivery service has its own alert file. New alerts are appended to the existing file — never create a new one unless none exists.

## Alert File Locations (Delivery)

| Service | Alert file |
|---------|-----------|
| `rt-surge-delivery` | `rt-surge/src/main/resources/moncon/umonitor-alert-group/delivery/rt-surge-delivery.yaml` |
| `rt-surge-compute-delivery` | `compute/src/main/resources/moncon/umonitor-alert-group/delivery/rt-surge-compute-delivery.yaml` |
| `surge-data-delivery` | `surge-data/src/main/resources/moncon/umonitor-alert-group/surge-data-delivery.yaml` |

All paths are relative to `~/Uber/fievel/marketplace/surge/`.

## Alert Entry Structure

Every alert entry in `spec.alerts` follows this shape:

```yaml
- template:
    $ref: file:///umonitor-alert-group/templates/delivery/<template-name>.yaml
  context:
    service: <service-name>
  alert:
    name: <Simple, descriptive alert name>
    description: <What is breaching, impact on eaters/restaurants/couriers, where to start investigating. 2-3 sentences for someone woken up at 3am.>
    query: |-
      <M3QL query>
    warn: <numeric threshold>
    critical: <numeric threshold>
    sustainPeriod: <seconds>
    contextGraphs:           # optional — diagnostic graphs shown in alert UI
      - name: <graph name>
        query: <M3QL>
    actions:                 # optional — overrides template actions
```

## rt-surge-delivery is Different

`rt-surge-delivery.yaml` uses **inline YAML anchors** (`*template`, `*templateZonal`) defined in its `container:` section — not `$ref:` files. Reference them directly:

```yaml
- template: *template       # LEGACY type — global alert
  alert:
    name: ...
    query: ...
    warn: 0.1
    critical: 0.15
    sustainPeriod: 600
    actions: *allActions    # pagerduty + email + udeploy
```

Or for per-zone alerting:
```yaml
- template: *templateZonal  # ZONE type — fires per availability zone
```

**Choosing urgency for rt-surge-delivery alerts:**

| Urgency | When to use | Actions anchor |
|---------|-------------|----------------|
| High (wake oncall) | Marketplace reliability, SLA breach, data loss | `*allActions` |
| Low | Experiment quality, data signals, non-SLA issues | `actions: pagerDutyActions: *pagerDutyActionLowUrgency` |

When in doubt, check analogous alerts already in the file for precedent.

## Template Reference (compute and surge-data)

Templates live in `templates/delivery/` within each service. Choose based on urgency and alert type:

| Template | When to use |
|----------|-------------|
| `adhoc-template.yaml` | Standard high-urgency alert (compute) |
| `adhoc-zonal-template.yaml` | Per-zone variant (compute) |
| `adhoc-template-require-manual-resolution.yaml` | Alert that needs human sign-off to close |
| `adhoc-high-urgency-business-hours.yaml` | High urgency, different PD key |
| `adhoc-low-urgency.yaml` (compute) | Low-urgency, no auto-resolve pressure |
| `adhoc.yaml` (surge-data) | Standard, includes uDeploy action |
| `adhoc-low-urgency.yaml` (surge-data) | Low-urgency, different PD service key |
| `adhoc-zonal.yaml` (surge-data) | Per-zone for surge-data |

**LEGACY vs ZONE:** `LEGACY` fires when the query threshold is breached globally. `ZONE` fires per availability zone — use when regional isolation matters.

## Required File Header

Every alert YAML must start with:

```yaml
dialect:
  slug: umonitor-alert-group
  version: v1

spec:
  group:
    name: <group-name>
    description: <description>
    email: ue-eater-rtp-eng@uber.com
    serviceName: <service-name>
    team:
      name: Eater Realtime Pricing
      email: ue-eater-rtp-eng@uber.com

  alerts:
    - ...
```

Since you're adding to an existing file, the header is already there — just append to `spec.alerts`.

## Common Fields

| Field | Notes |
|-------|-------|
| `warn` / `critical` | Numeric threshold matching what the M3QL query returns |
| `sustainPeriod` | Seconds the threshold must be breached before firing. Use ≥300 for transient-safe, ≥600 for sustained issues |
| `intervalSeconds` | Evaluation interval — set in template (usually 120), don't override |
| `query` | M3QL; **always** filter `flow_id:primary` — shadow flows will pollute the signal. Do not add `environment_type` as a filter. |

## Default Urgency

Unless the user specifies otherwise, **default to high urgency**:
- For `rt-surge-delivery`: use `*allActions` (with `*template` for global, `*templateZonal` for zonal)
- For compute/surge-data: use `adhoc-template.yaml` for global, `adhoc-zonal-template.yaml` for zonal

## Before Writing the Alert

Before editing the YAML file, **explain the alert in plain English** to the user:
1. What the alert tests (in words, not M3QL)
2. Proposed thresholds and sustain period
3. Ask if the alert should be changed somehow

Only write the YAML after the user confirms.

## Context Graph Best Practices

Context graphs help oncall quickly identify the source of an issue. Follow these patterns:

- **Always add at least one context graph** — the raw metric broken down by the most useful dimension
- **Use `| head 5`** to show only the top 5 offenders — avoids overwhelming the graph with hundreds of series
- **Common breakdowns for Delivery alerts:**
  - By city: break down by `region_id` using the `_city_level` rollup variant if available
  - By DC: break down by `dc`
  - By step/component: break down by the tag that the alert groups on
- **Use `| head 5` for top offenders, `| tail 5` for bottom offenders** (e.g. drop-below alerts)

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `$ref:` in rt-surge-delivery.yaml | Use inline anchor `*template` or `*templateZonal` instead |
| Forgetting `context.service` field | Required when using `$ref:` templates — it sets `${service}` in template links |
| Creating a new alert file | Append to the existing delivery alert file for the service |
| Not filtering `flow_id:primary` in query | Shadow flows pollute signal |
| Mismatched `warn`/`critical` types | Must be numbers, not strings |
| Using implementation details in alert names | Use domain terms and gRPC endpoints, but be specific — "DZR Docstore Cache Refresh Staleness" not "DZR Stale Data" |
| Missing or vague alert description | Description must include: what is breaching, impact on eaters/restaurants/couriers, and where to start investigating |
