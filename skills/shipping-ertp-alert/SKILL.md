---
name: shipping-ertp-alert
description: Use when an ERTP alert YAML is written and tested and ready to be submitted as a Phabricator diff for review
---

# Shipping an ERTP Alert

## Overview

Once the alert YAML is written and tested, ship it by: generating a moncon preview link, creating a diff with a structured summary, and linking the moncon preview in the test plan.

## Required Inputs

- Path to the alert YAML file that was modified (e.g. `rt-surge/src/main/resources/moncon/umonitor-alert-group/delivery/rt-surge-delivery.yaml`)
- Alert intent (what the alert is for, 1–2 sentences)
- Breach history from `testing-ertp-alerts` (current value, 7d/14d critical breaches)

## Step 1: Apply moncon in staging

**Run `moncon apply` in staging every time before creating or updating a diff.** This validates the YAML and generates a preview link:

```bash
cd ~/Uber/fievel/marketplace/surge/rt-surge/src/main/resources/moncon

# Apply in staging (default, no -e flag)
echo "y" | moncon apply -i <relative-path-to-alert-file>
# e.g. echo "y" | moncon apply -i umonitor-alert-group/delivery/rt-surge-delivery.yaml
```

The command outputs a staging moncon URL — copy it. This is the test plan.

## Step 2: Create or update the diff

Use `uber-dev:diff-create` for new diffs or `uber-dev:diff-update` when adding alerts to an existing diff. **Always update the diff description when adding new alerts.**

**Summary format — one section per alert:**
```
## Alert: <simple alert name>

<1–2 sentence description of what the alert detects and why it matters.>

Breach history:
- Current value: <X>
- 14-day peak: <X>
- 7-day critical breaches: <N> (~<M> sustained fires)
- 14-day critical breaches: <N> (~<M> sustained fires)
```

Repeat the above block for each alert in the diff.

**Test Plan — link only:**
```
<moncon staging URL from Step 1>
```

Do not add any other text to the test plan. Just the link.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Running `moncon apply` from the wrong directory | Must run from the `moncon/` folder of the service, not the repo root |
| Forgetting the moncon URL in the test plan | Reviewers need it to preview the alert in the moncon UI before approving |
| Not including breach history in summary | Reviewers need to know the alert won't be immediately noisy |
| Adding text besides the URL in the test plan | Test plan should contain only the moncon staging link |
| Not updating diff description when adding more alerts | Every alert in the diff needs its own section in the summary |
| Not running moncon apply before updating an existing diff | Always re-run moncon apply before every diff create or update |
