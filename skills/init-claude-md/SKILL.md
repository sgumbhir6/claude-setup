---
name: init-claude-md
description: Use when a user wants to auto-generate CLAUDE.md files for a directory tree, initialize context files for a new codebase or folder, or populate every subfolder with a CLAUDE.md starting from a root directory
---

# Init CLAUDE.md

## Overview

Spawns a subagent that walks a directory tree top-down and generates a contextual CLAUDE.md in each subfolder. Parent folders are processed before children so each level can reference parent context.

## When to Use

- User says "init CLAUDE.md", "generate CLAUDE.md files", "create CLAUDE.md recursively"
- Setting up Claude context for a new repo or folder structure
- Onboarding Claude to an unfamiliar codebase

## Process

### Step 1: Announce and confirm root

Tell the user the root directory you'll use (default: current working directory). If ambiguous, ask.

### Step 2: Spawn a general-purpose subagent

Pass it the full instructions below. Do not do this work inline — delegate entirely.

---

**Subagent prompt template:**

```
You are initializing CLAUDE.md files for a directory tree.

Root directory: <ROOT_PATH>

Instructions:

1. Use Glob with pattern `**/` to discover all subdirectories under the root.
   Exclude any paths containing: node_modules, .git, __pycache__, .venv, dist, build, target, .next, out, coverage

2. Sort directories by depth (shallowest first — process parents before children).

3. For each directory in order:
   a. Check if CLAUDE.md already exists in that directory. If yes, skip it.
   b. Use Glob to list all files directly in that directory (non-recursive).
   c. Read key files to understand the folder's purpose:
      - README*, CLAUDE.md in parent, index.*, main.*, __init__.py, package.json, build.gradle, pom.xml, go.mod
      - Up to 3-5 representative source files
   d. Read the parent directory's CLAUDE.md (if it exists) for context.
   e. Write a CLAUDE.md to the directory with this structure:

      # <Folder Name>

      ## Purpose
      <1-2 sentences: what this folder is for>

      ## Contents
      <Bullet list of key files/subfolders and what they do>

      ## Patterns & Conventions
      <Any notable patterns, naming conventions, or practices observed>

      ## Working Here
      <Practical notes a developer needs when working in this folder>

   Keep each CLAUDE.md concise: 30-100 lines. Be specific, not generic.

4. After processing all directories, output a summary:
   - Folders where CLAUDE.md was created
   - Folders skipped (already had CLAUDE.md)
   - Any folders skipped because content was unclear
```

---

### Step 3: Report back

Relay the subagent's summary to the user.

## Rules

- **Top-down only** — never process a child before its parent
- **Never overwrite** existing CLAUDE.md files unless user explicitly says `--overwrite`
- **Skip noise folders**: node_modules, .git, __pycache__, .venv, dist, build, target
- **Be specific** — generic CLAUDE.md content is worse than none

## Options

| Flag | Behavior |
|------|----------|
| `--overwrite` | Regenerate CLAUDE.md even if one exists |
| `--depth N` | Only go N levels deep (default: unlimited) |
| `--root <path>` | Use a specific root instead of cwd |
