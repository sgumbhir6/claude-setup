---
name: reading-plans
description: Use when there is any possibility that prior investigation, context, or planning work exists in ~/plans/ that could inform the current task — before writing proposals, starting investigations, writing code, or answering questions about known problems. Bias toward invoking rather than not.
---

# Reading Plans

## Overview

The `~/plans/` directory stores structured context from prior investigations. Before starting work that might benefit from this context, read `index.md` to discover what exists, then read only the file type appropriate for your current purpose.

**Do not read all files. Read only what your purpose requires.**

## When to Invoke

Invoke this skill if ANY of these apply:
- Starting an investigation that might overlap with prior work
- Writing a proposal, doc, or stakeholder communication
- About to write code or spawn a coding agent on a known problem area
- Answering a question about a system that may have been previously investigated
- The user mentions a problem name, symptom, or service that might have a plan

**When in doubt, invoke.** The cost of checking is one file read. The cost of missing context is rebuilding work that already exists.

## Step 1: Read the Index

```
~/plans/index.md
```

Each entry is: `folder-name | 5-word description`

If `index.md` does not exist, glob `~/plans/` to discover what's there.

Identify which plan folder(s) are relevant to your current task.

## Step 2: Read the Right File

Read **only** the file type that matches your current purpose. Do not read files you don't need.

| Purpose | File to read |
|---------|-------------|
| Understanding the problem — what is being investigated, scope, services affected, constraints | `~/plans/<name>/agent-context.md` |
| Writing a proposal, doc, or communication for human stakeholders | `~/plans/<name>/problem-statement.md` |
| Writing a business doc that needs evidence to support recommendations | `~/plans/<name>/problem-statement.md` + `~/plans/<name>/code-exploration/findings.md` |
| Reviewing what code was already investigated — evidence, call chains, confirmed paths | `~/plans/<name>/code-exploration/findings.md` |
| Writing code or spawning a coding agent — need verified file paths, class names, data flows | `~/plans/<name>/code-exploration/agent-code-context.md` |
| Understanding technical constraints for a business doc section | `~/plans/<name>/agent-context.md` |

### File type guide

**`agent-context.md`** — technical problem context. Read when you need to understand what problem is being worked on, what services and repos are in scope, what the key entry points are, and what constraints apply. This is the starting point for any new agent working on the problem.

**`problem-statement.md`** — human-readable framing. Read when producing output for human stakeholders: proposals, team docs, incident summaries, planning documents. Contains context, problem statement, and optional proposed solutions in plain language.

**`code-exploration/findings.md`** — code investigation evidence. Read when you need to know what was already found in the code: specific failure paths, verified behavior with file:line refs, confirmed/ruled-out hypotheses. Avoid re-exploring code that's already been investigated.

**`code-exploration/agent-code-context.md`** — verified code facts for agents. Read when you are about to write code, create a fix, or spawn an agent that needs to work with specific code. Contains exact file paths, class/function names with line numbers, data flow, and observed patterns — all verified by prior exploration.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Reading all files in the plan | Read only what your purpose requires |
| Reading `agent-context.md` to write a stakeholder doc | Use `problem-statement.md` for human output |
| Reading `problem-statement.md` to write code | Use `agent-code-context.md` for coding tasks |
| Skipping this skill because "the task is new" | Prior work may overlap. Check `index.md` first. |
| Skipping because no index.md exists | Glob `~/plans/` instead — plans may still exist |
