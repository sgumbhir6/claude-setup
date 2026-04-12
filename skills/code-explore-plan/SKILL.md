---
name: code-explore-plan
description: Use when unfamiliar with a codebase area, unclear on change impact, or needing code-level evidence to back up a problem statement or discover answers to technical questions before proposing changes
---

@../explore-problem/SKILL.md

# Code Explore Plan

## Overview

Systematically explore code paths to back up a problem statement with evidence, or discover answers to technical questions. **Every claim must be grounded in code you actually read — no assumptions, no hallucination, no inferences from naming conventions.**

**This skill always loads `explore-problem`** (which loads `clarify-requirements`). The folder `~/plans/<problem-name>/` and `agent-context.md` must exist before the exploration agent is spawned. If they do not exist, run `explore-problem` first.

## The Iron Rule: Evidence Only

**You are NOT allowed to:**
- State anything about code you have not directly verified with Read, Grep, or Glob
- Infer behavior from class names, method names, or patterns
- Write findings that are not anchored to a specific `file:line` reference
- Skip creating `agent-code-context.md` because the task "only asked for findings"

**When you cannot find something:** say so explicitly. "I could not find this file/class/method" is a valid and valuable finding. A confirmed absence beats a guessed answer.

## Process

### Step 1: Verify Prerequisites

Read `~/plans/<problem-name>/agent-context.md`. If it does not exist, **STOP**. Do not proceed.

**You are NOT allowed to substitute:**
- Context from the conversation or user's message
- Context from memory or prior knowledge
- Any inline context provided alongside this skill

The file must exist on disk. If it does not, tell the user and ask them to run `explore-problem` first. No exceptions.

### Step 2: Spawn Exploration Agent

Use `Agent` tool (`subagent_type="general-purpose"`) with the prompt template below.

**Output — both files are REQUIRED:**
- `~/plans/<problem-name>/code-exploration/findings.md`
- `~/plans/<problem-name>/code-exploration/agent-code-context.md`

Missing either file is a failure.

## Quick Reference

| Phase | Goal |
|-------|------|
| Verify | `agent-context.md` must exist |
| Explore | Every claim verified with Read/Grep/Glob |
| `findings.md` | Human-readable: questions answered with evidence |
| `agent-code-context.md` | Agent-readable: structured facts for downstream agents |

## Agent Prompt Template

```
You are conducting systematic code exploration.

## YOUR FIRST STEP: Read context files

Read BOTH of these before touching any code:
1. ~/plans/{PROBLEM_NAME}/agent-context.md
2. ~/plans/{PROBLEM_NAME}/problem-statement.md

If either file does not exist, stop and report this immediately.

## ABSOLUTE RULE: EVIDENCE ONLY

You MUST NOT:
- State anything about code you have not directly verified with Read, Grep, or Glob
- Infer behavior from class or method names
- Hallucinate method signatures, class hierarchies, or data flows
- Include any finding not anchored to a specific file:line reference you actually read

When you cannot find something, write: "Not found: [what you searched for, what you tried]"
This is a valuable finding. Do not substitute a guess.

## SCOPE CONSTRAINT

Answer only the questions in the problem statement. If you find adjacent issues:
- Do NOT include them in the output files
- List them after the files and ask: "I also found [X]. Include it?"

## Explore

1. Start from entry points in agent-context.md
2. For each entry point: read the file, trace execution flow, follow call chains
3. For every claim you want to make, find the exact file:line that supports it
4. If you cannot find support — do not make the claim

## Write TWO Output Files

Create `~/plans/{PROBLEM_NAME}/code-exploration/`.

**Both files are required. Do not skip either one.**

### findings.md — for humans

- **Questions**: Restate each question from the problem statement, then answer it with evidence
- **Evidence**: Every claim uses `path/to/file.ext:line` with a code snippet
- **Execution Flow**: Actual call chain found (not inferred)
- **Impact**: What components are affected, with refs
- **Not Found**: What you searched for and could not find — be explicit

### agent-code-context.md — for downstream Claude agents

A complete code-level context package. Future agents will have NO other context.

- **Relevant Files**: Full paths of every file read, with purpose
- **Key Classes / Functions**: Name, `file:line`, verified behavior
- **Data Flow**: How data moves through these code paths (with refs)
- **Entry Points**: Where to start for this problem
- **Observed Patterns**: What you actually saw in the code (not inferred conventions)
- **Constraints**: What must not change, what must be preserved
- **Open Questions**: What still needs investigation

## Your Task

**Problem Name:** {PROBLEM_NAME}
**Context:** ~/plans/{PROBLEM_NAME}/agent-context.md
**Problem Statement:** ~/plans/{PROBLEM_NAME}/problem-statement.md

Steps:
1. Read both context files
2. List the specific questions to answer
3. Explore with Read, Grep, Glob — verify every claim
4. Write BOTH files to ~/plans/{PROBLEM_NAME}/code-exploration/
5. List any adjacent findings at the end
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping `agent-code-context.md` | **Both files are mandatory.** "The task only asked for findings" is not a valid reason. |
| Making claims without `file:line` | Find the line or don't make the claim |
| Exploring before `agent-context.md` exists | Run `explore-problem` first |
| Writing "not found" as a guess | State exactly what you searched for and what tools you used |
| Scope creep | Adjacent findings → list at end → ask before including |
| Both files having the same content | `findings.md` = human narrative with evidence. `agent-code-context.md` = structured facts for agents. |
