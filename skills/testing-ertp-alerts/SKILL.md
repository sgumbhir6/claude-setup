---
name: testing-ertp-alerts
description: Use when manually testing an ERTP alert before shipping — runs the M3QL query against historical data and reports how many times warn/critical thresholds would have been breached over the last 7 and 14 days
---

# Testing ERTP Alerts Manually

## Overview

Before shipping an alert, verify it's well-calibrated: not too noisy (fires constantly) and not too quiet (never fires). Use the alert's own M3QL query against historical production data to count threshold breaches.

## Required Inputs

You must have:
- `query` — the M3QL query from the alert YAML
- `warn` — warn threshold value
- `critical` — critical threshold value
- `sustainPeriod` — seconds the threshold must be sustained before firing

## Step 1: Get current value and 14-day range

Run the alert query over 14 days to see the full time series shape:

```
# m3ql_query_metrics tool
start_time: 14 days ago (RFC3339)
end_time: now
query: <paste alert query exactly>
```

From the returned data points note:
- **Current value** (most recent point)
- **Peak value** (max across the window)
- **Typical range** (rough min–max)
- Shape: flat, spiky, trending up/down

If the peak is already below the warn threshold → **alert is too quiet, stop here** and recommend lower thresholds.

## Step 2: Count critical breaches (7 and 14 days)

Find the peak over 7d and 14d:

```
<original query>
| moving 7d max

<original query>
| moving 14d max
```

Count data-point intervals above the critical threshold:

```
<original query>
| > <critical_threshold>
| sumSeries
| summarize 7d sum
```

Each count is a number of 120-second intervals above critical. To estimate **sustained alert fires**:
```
sustained_fires ≈ (breach_intervals × 120) / sustainPeriod
```

Repeat with `summarize 14d sum` for the 14-day count.

## Step 3: Assess calibration

| Result | Assessment | Action |
|--------|-----------|--------|
| 0 critical fires in 14 days, metric is a steady-state signal | Too quiet — thresholds may be too high | Verify thresholds against peak |
| 0 critical fires in 14 days, metric is an error/incident-only signal | Expected and correct — these alerts are designed to be silent | Ship as-is |
| > 3 critical fires/week | Too noisy | Raise threshold or increase `sustainPeriod` |
| Current value near critical threshold | Will fire immediately on deploy | Verify this is a real issue or raise threshold |

**How to tell the difference:** If the metric is a counter that only fires on error conditions (misconfiguration, fallback, model missing), 0 fires in 14 days is correct behavior. If the metric is a ratio or rate that should always have a value (e.g. request rate, header missing %), 0 fires means the threshold is set too high.

## Step 5: Report findings

Output a summary:

```
Current value: <X>
14-day peak: <X>
7-day critical breaches: <N> (~<M> sustained fires)
14-day critical breaches: <N> (~<M> sustained fires)
Assessment: <too noisy | too quiet | well-calibrated>
Recommendation: <ship as-is | adjust critical to X | increase sustainPeriod to Xs>
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Running query without `flow_id:primary` filter | Alert query should already have this — if it doesn't, the test data is wrong |
| Judging calibration on current value alone | Always check 7d and 14d history |
| Forgetting `sustainPeriod` when estimating fires | A 1-hour breach at `sustainPeriod: 600` fires ~6 times, not 1 |
| Testing during an incident window | Incident data will make a good alert look noisy — note the date range |
