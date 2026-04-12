---
name: creating-ertp-alert
description: Use when asked to create an ERTP Delivery surge alert end to end — from understanding the metric through to shipping the diff
---

# Create an ERTP Alert End to End

## Overview

Creating an ERTP alert requires four sequential steps. Each step has a dedicated skill — invoke them in order. Do not skip steps or reorder them.

## Sequence

### Step 1: Understand the metric
**REQUIRED SKILL: understand-ertp-metrics**

Before writing any alert, find the metric in code, confirm its full M3 name, understand what triggers it, what tags it carries, and verify it exists in production M3.

Do not proceed to Step 1.5 until you can answer:
- What is the full M3 metric name?
- What does it measure?
- Which Delivery service emits it?

---

### Step 1.5: Check if the alert already exists

Before writing anything, search the target alert file for the metric name or alert name:

```bash
grep -n "<metric_name>\|<alert_name>" <path-to-alert-yaml>
```

If a matching alert already exists — **stop**. Do not create a duplicate. Review the existing alert instead and proceed to Step 3 to validate its calibration if needed.

---

### Step 2: Write the alert YAML
**REQUIRED SKILL: adding-ertp-moncon-alerts**

Add the alert to the correct moncon file for the service. Use `samples.md` to choose the right alert type (error rate, WoW, ratio, etc.) and template. Always filter `flow_id:primary` in the query.

**Before editing the file**, explain to the user in plain English:
1. What the alert tests
2. Proposed thresholds and sustain period
3. Ask if the alert should be changed somehow

Only write the YAML after the user confirms.

**Output: the moncon YAML file edited with the new alert appended.**

---

### Step 3: Test calibration
**REQUIRED SKILL: testing-ertp-alerts**

Run the M3QL query over 14 days. Count critical breaches over 7d and 14d. Assess whether the thresholds make sense given the observed data. Adjust if the alert would fire immediately on deploy.

---

### Step 3.5: Ask about additional alerts

After calibration passes, ask the user: **"Do you want to add any more alerts to this diff before shipping?"**

If yes, repeat Steps 2–3 for each additional alert. Only proceed to Step 4 when the user confirms no more alerts are needed.

---

### Step 4: Ship the diff
**REQUIRED SKILL: shipping-ertp-alert**

**Run `moncon apply` in staging** every time before creating or updating a diff. Then:
- Summary: one section per alert — alert name, intent, breach history (see shipping-ertp-alert for format)
- Test plan: moncon staging URL only (just the link, nothing else)
- When adding alerts to an existing diff, **always update the diff description** to include the new alert sections

## Alert Naming

Use clear, descriptive names that tell oncall what is wrong and where to start looking. Domain terms (BAF, DZR, ETD surge, Laminator, EATS_LITE) and gRPC endpoints are fine. The name should be specific enough that oncall doesn't need to read the query to understand the alert.

| Good | Bad |
|------|-----|
| "Delivery Surge Latency p99" | "Delivery Surge Slow" |
| "BAF Computation Error Rate" | "BAF Errors" |
| "DZR Docstore Cache Refresh Staleness" | "DZR Stale Data" |
| "ETD Surge getDynamicSurgeDecision Error Rate" | "ETD Surge Errors" |

## Alert Description

Every alert must have a `description` field that helps oncall understand:
1. **What this alert means** — what is breaching and why it matters
2. **Impact** — what happens to eaters/restaurants/couriers if this stays breached
3. **Where to start** — first thing to check (logs, dashboard, upstream service)

Example:
```yaml
description: >-
  BAF computation error rate exceeded threshold. Eaters may see
  incorrect or missing surge fees. Check rt-surge-delivery logs
  filtered by flow_id:primary for BAF computation exceptions.
```

Keep it to 2-3 sentences. Write for someone woken up at 3am.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping Step 1 and guessing the metric name | Always verify the full M3 name from code before writing the query |
| Skipping Step 3 | Thresholds that look right may fire immediately or never — always check |
| Running moncon apply from the wrong directory | Must run from the service's `moncon/` folder |
