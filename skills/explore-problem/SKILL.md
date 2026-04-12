---
name: explore-problem
description: Use when starting any investigation, planning task, or work that requires building shared understanding of a problem — before any analysis, exploration, or code reading begins
---

@../clarify-requirements/SKILL.md

# Explore Problem

## Overview

**STOP. Before you do anything else — before reading code, before analyzing, before exploring — you must complete this skill.**

This skill's ONLY job is to:
1. Clarify the problem completely with the user
2. Write two structured files to `~/plans/<problem-name>/`

That's it. No analysis. No code reading. No exploration. Those come later via other skills.

**This skill always loads `clarify-requirements`.** Follow it fully.

## The Iron Rule

**You are NOT allowed to:**
- Read any code before completing this skill
- Provide analysis or insights before completing this skill
- Assume you understand the problem and skip to file creation
- Create the files without confirmed alignment on the problem
- Ask multiple clarifying questions at once

If you are tempted to "just quickly look at the code first" or "share what you already know" — **STOP. That is a violation. Follow this skill.**

## Red Flags — You Are Rationalizing

| Thought | Reality |
|---------|---------|
| "I already know enough to help — let me share what I know" | That is analysis. This skill runs before analysis. Stop. |
| "Let me just share some initial thoughts, then clarify" | No. Clarification comes before everything else. |
| "The problem is clear, clarification would waste time" | Every investigation has hidden assumptions. Surface them. |
| "I'll create the files with what I know, then refine" | Files created without confirmed alignment are wrong. Confirm first. |
| "I'll ask all my questions at once to save time" | Users skip questions, give partial answers. One at a time. Always. |
| "The user gave me enough detail, I don't need to ask more" | You have assumptions you haven't surfaced. Ask. |

## Process

### Phase 1: Restate & Clarify (follow clarify-requirements exactly)

1. Restate the problem in your own words — be specific
2. Identify every assumption you are making
3. Ask ONE clarifying question at a time, wait for the answer
4. Keep asking until the user explicitly confirms the problem statement is complete and correct

**Do not proceed to Phase 2 until the user has confirmed.**

Common questions to ask (one at a time):
- What service or system is affected?
- What have you already ruled out or tried?
- Are there specific entry points you know are relevant?
- What does "done" look like for this investigation?
- What should NOT be explored?

### Phase 2: Ask for Problem Name

Once aligned, ask: "What should I call this problem folder? (e.g. `surge-zero-peak`, `auth-flow-refactor`)"

Wait for the answer.

### Phase 3: Write Two Files and Update Index

Create `~/plans/<problem-name>/` and write both files. **Both are required. Missing either is a failure.**

After writing the files, update `~/plans/index.md` by appending:
```
<problem-name> | <5-word description of the problem>
```

If `~/plans/index.md` does not exist, create it with a header line first:
```
# Plans Index
# format: folder-name | 5-word description

<problem-name> | <5-word description>
```

#### `agent-context.md`

Context for another Claude agent that has NO other information. Be explicit and complete.

```markdown
# Agent Context

## Project / Repo
[Repo name, key directory paths, language, framework]

## Team / Domain
[Team, service or system affected]

## Problem Summary
[2–3 sentences: what is wrong or needs to happen, in technical terms]

## Key Starting Points
[Exact file paths, class names, function names, API endpoints — be specific]

## Known Constraints
[What must not change. What is out of scope. Patterns to follow.]

## What the Agent Should Do
[Clear, unambiguous instruction for the agent]

## What the Agent Should NOT Do
[Scope boundaries. What to leave alone.]
```

#### `problem-statement.md`

For human stakeholders. Keep technical jargon minimal.

```markdown
## Context
[Background — what system, what situation, why this matters]

## Problem Statement
[What specifically is wrong or needs to happen. Be concrete.]

## Proposed / To-Be-Explored Solutions *(optional)*
[Known approaches, or note that exploration is the next step]
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Providing analysis before clarifying | This is a VIOLATION. Stop. Follow Phase 1 first. |
| Skipping clarification because "the problem is clear" | You have assumptions. Surface them. |
| Creating files before the user confirms the problem | Confirmation is required before Phase 3. |
| Asking multiple questions at once | One at a time. Always. |
| Vague `agent-context.md` | The downstream agent has zero context — include exact paths, names, explicit instructions. |
| Mixing stakeholder and technical detail | `problem-statement.md` = human-readable. `agent-context.md` = technical. |
| Only creating one file | **Both files are required. Missing either is a failure.** |
