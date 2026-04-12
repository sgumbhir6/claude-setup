---
name: write-business-doc
description: Use when creating a business-facing document (one-pager, proposal, investigation summary, headrooming) from prior investigation artifacts in ~/plans/ — transforms technical findings into stakeholder-ready documents
---

@../reading-plans/SKILL.md

# Write Business Doc

## Overview

Generate business-facing documents from investigation artifacts in `~/plans/<problem-name>/`. Each document type has a defined template with explicit stakeholders, sections, and guidance on which plan files to read for each section.

**This skill always loads `reading-plans`.** Follow it to discover and read the right artifacts before writing.

## Process

### Step 1: Verify Prerequisites

Check that `~/plans/<problem-name>/` exists and contains investigation artifacts. If it does not exist, tell the user and stop.

### Step 2: Identify Document Type

Ask the user which document type to create. Load the corresponding template from `templates/`.

Available templates:
- `templates/one-pager-proposal.md` — Proposal for engineers + PMs
- `templates/one-pager-investigation.md` — Investigation summary for engineers
- `templates/one-pager-headrooming.md` — Solution exploration for engineers + PMs

### Step 3: Read Template

Read the template file. It defines:
- **Stakeholders** — who the document is written for
- **Sections** — exact structure with descriptions
- **Source mapping** — which plan file(s) to read for each section (use `reading-plans` to read them)
- **Tone guidance** — how to calibrate language for the audience

### Step 4: Ask User for Sub-Sections (if template requires)

Some templates have sections where the user defines the sub-headings (e.g., "Proposed Solution" in a proposal). When the template marks a section as `[ask user for sub-sections]`, ask the user what sub-headings to include before writing.

### Step 5: Write Draft

Write the document following the template structure exactly. Save as:
```
~/plans/<problem-name>/<document-type>.md
```

Naming convention:
- `proposal.md`
- `investigation.md`
- `headrooming.md`

### Step 6: User Review

Present the draft to the user for review. Make changes as requested. Do not proceed until the user approves.

### Step 7: Publish to Google Docs

After user approval, use the google-mcp to create a Google Doc with the approved markdown content.

## Writing Principles

- **Concise and clear** — one-pagers don't have to be exactly 1 page but should be scannable
- **Lead with "why it matters"** — not implementation details
- **Match tone to stakeholders** — if PMs are in the audience, minimize code-level jargon; use service names and business impact
- **Evidence over assertions** — reference investigation findings, don't make unsupported claims
- **No invented sections** — follow the template exactly, do not add TL;DR, Risks, Timeline, or other sections unless the template includes them

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Reading all plan files upfront | Use `reading-plans` — read only what each section's source mapping specifies |
| Inventing sections not in the template | Follow the template structure exactly |
| Writing for the wrong audience | Check the template's stakeholders field |
| Skipping the sub-section ask for proposals | If template says `[ask user]`, ask before writing |
| Saving with inconsistent filename | Use the naming convention: `proposal.md`, `investigation.md`, `headrooming.md` |
| Skipping Google Docs publish | Always publish after user approval |
| Including code snippets or file paths | Business docs reference services and behaviors, not code |
