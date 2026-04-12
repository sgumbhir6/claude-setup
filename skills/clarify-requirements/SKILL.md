---
name: clarify-requirements
description: Use when planning, problem-solving, investigating, or making any code change — even if the goal seems clear. Invoke whenever the user describes a problem to solve, proposes a change, or asks to build/fix/modify something. Also invoke for investigations ("check if", "look into"), incident response, and opinion requests.
---

# Clarify Requirements

<EXTREMELY-IMPORTANT>
If you are about to build, implement, fix, or change something — and you have not confirmed every assumption with the user — you MUST follow this skill first.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## The Rule

**Stop. Gather requirements. Resolve every assumption. Get explicit confirmation. Only then start work.**

Incomplete requirements produce wrong solutions. Clarifying upfront costs minutes. Fixing wrong work costs hours.

## The Iron Law

You are not allowed to:
- Assume you already understand and start working
- Ask all clarifying questions at once to "save time"
- Start with what you know and clarify later
- Reason your way to a confident interpretation and proceed
- Treat this as optional because the request "seems clear"
- **Substitute a different solution for what the user asked** — if the user asks for X, clarify X. Do not propose Y as "simpler" before understanding what they actually want. You can mention alternatives _after_ you understand the requirements, not instead of gathering them.

If you are thinking any of the thoughts in the Red Flags section below — **STOP**. You are rationalizing. Follow the skill.

## When to Invoke

**Always invoke when:**
- You are asked to build, implement, fix, or change something
- The request involves behavior you need to understand (not just mechanical execution)
- You are unclear on scope, constraints, or expected outcome
- The user has not specified exactly what "done" looks like
- You are making any assumption about intent
- **The user gives a high-level design or restructuring but has not specified the details** — e.g., "split X into A and B", "refactor this into components", "redesign how X and Y relate"
- **How to implement something is not fully specified** — the implementation choices ARE the requirements
- **The user asks you to investigate, check, or verify something** — "check if X is correct", "look into why Y", "verify Z". Investigations have implicit requirements: what does "correct" mean? What counts as an answer? What scope should you cover? These are requirements you need to confirm.
- **The user describes an incident or urgent problem** — "X is broken", "need this fixed ASAP", "production issue". Urgency does NOT exempt you from clarifying. Wrong fixes under time pressure are worse than asking one question first. Even in incidents, you need to know: what is the symptom exactly? What changed? What is the expected behavior?
- **The user asks for your opinion or assessment** — "tell me what you think", "look at X and let me know", "what do you think about Y". Opinion requests have hidden requirements: what aspect should you evaluate? What criteria matter to the user? What would be actionable feedback vs. noise?

**Invoke even at 1% uncertainty.** The cost of invoking and discovering you didn't need it is low. The cost of skipping and building the wrong thing is high.

**You do NOT need explicit permission to invoke this skill.** The user asking you to do something is sufficient trigger.

## Process

### Step 1: Restate Your Understanding

Before asking anything, restate what you understand the request to be in your own words. Be specific — name the thing being built, the problem being solved, the behavior expected.

> "Let me make sure I understand: you want X, which should do Y, because Z."

This surfaces misunderstandings immediately and shows the user what you're working from.

### Step 2: Identify All Assumptions

Before asking the user anything, list internally every assumption you are making. These become your clarifying questions. Common assumptions:
- What is the expected input/output or behavior?
- What are the constraints or edge cases?
- What does "done" look like?
- What should NOT happen?
- Are there existing patterns or conventions to follow?
- Who is the user of this feature/fix?
- **Where should this live?** — file location, directory, global vs project-scoped
- **What is the source of truth?** — if syncing or mirroring, which copy is authoritative?
- **What is the direction of data flow?** — one-way vs bidirectional, push vs pull
- **What is the scope?** — all items vs specific items, automatic vs manual
- **What happens on conflict or overlap?** — overwrite, skip, warn, merge

### Step 3: Read Relevant Code and Context (Quick Scan)

Before asking questions, do a **fast, targeted read** of the code areas most likely involved in the change. The goal is not deep understanding — it's to ask **informed** questions instead of naive ones.

- **Check CLAUDE.md files** in relevant directories first — they document patterns, conventions, and architecture that inform your questions
- Use Glob/Grep to find the key files (classes, methods, configs) related to the request
- Read the most relevant 1–3 files (or sections of files) — no more
- Spend no more than a minute or two here — this is a scan, not an investigation
- Note: what patterns exist, what's already there, what might be affected

This makes your clarifying questions grounded in the actual code, not generic. For example, instead of "where should this go?" you can ask "should this go in `FooService` alongside the existing `bar()` method, or somewhere else?"

### Step 4: Ask ONE Question at a Time

Ask your most important clarifying question. Wait for the answer. Update your understanding. Then ask the next question if needed.

**Never batch questions.** Users skip questions, give partial answers, or get overwhelmed. One question forces a complete answer.

### Step 5: Present Final Understanding and Confirm

After resolving all assumptions, present your **complete understanding** of the change — not just a one-liner, but the full picture:

> **Here's my understanding of what we're doing:**
> - **What:** [specific change being made]
> - **Where:** [files/classes/methods affected]
> - **How:** [approach — key implementation details]
> - **What "done" looks like:** [expected outcome / acceptance criteria]
> - **What won't change:** [explicit scope boundaries, if relevant]
>
> **Does this capture it correctly?**

**Wait for an explicit "yes" before proceeding.** Do not start work, invoke other skills, or dispatch agents until the user confirms. Silence is not confirmation. A partial answer means ask again.

### Step 6: Start Work

Only now do you start.

## Red Flags

These thoughts mean STOP — you're rationalizing:

| Thought | Reality |
|---------|---------|
| "The request is clear enough" | You have assumptions. Surface them. |
| "I'll clarify as I go" | You'll build the wrong thing first. |
| "Asking feels like wasted time" | Wrong work wastes more time. |
| "I can infer what they want" | Inference is assumption. Confirm it. |
| "This is a simple task" | Simple tasks have implicit requirements too. |
| "They'll tell me if I'm wrong" | They expect you to ask, not guess. |
| "I don't want to slow them down" | Delivering wrong work is slower. |
| "I mostly understand it" | Mostly is not confirmed. |
| "I know the goal — I just need to figure out how to implement it" | The implementation choices ARE the requirements. How something is structured or split is what the user needs to confirm. |
| "Another skill already gives me a process to follow" | That process still depends on requirements you haven't confirmed. Load the process skill after clarifying. |
| "The user gave me names and a brief description, that's enough" | Names and one-liners leave design decisions implicit. What goes where? How do they interact? Confirm it. |
| "There's a simpler way to do this" | The user asked for X, not your alternative. Clarify X first. |
| "I know where this should go" | Location, scope, and ownership are requirements. Ask. |
| "The obvious default is fine" | Direction, scope, and trigger are choices the user must make. |
| "This is an investigation, not a development task" | Investigations have requirements too: what does "correct" mean? What scope should you cover? What counts as an answer? Clarify before diving in. |
| "This is a production incident — urgency means skip clarification" | Wrong fixes under time pressure are worse than one question. Even in incidents, you need: what exactly is the symptom? What changed? What is expected behavior? One focused question costs seconds; wrong investigation direction costs the real time. |
| "This is read-only / I'm just looking at code" | Reading code without knowing what the user considers relevant, correct, or important means you'll report on the wrong things. What aspect matters? What criteria? What would be actionable? |
| "The user said 'check if' or 'verify' — the task is self-evident" | "Check if X is correct" hides requirements: correct by what definition? Compared to what baseline? For which cases? Surface the implicit standard before verifying against it. |
| "I can start gathering data and ask questions as they come up" | You'll go down the wrong path first. One question upfront saves a round trip of wasted investigation. |

## Quick Reference

| Phase | Action | Rule |
|-------|--------|------|
| Restate | Say what you think you're doing | Before asking anything |
| Identify | List all assumptions internally | Be exhaustive |
| Quick Scan | Read 1–3 relevant files/sections | Fast scan, not deep dive — grounds your questions in actual code |
| Ask | One question at a time | Wait for full answer before next |
| Present & Confirm | Show full understanding, get explicit yes | Do NOT proceed until confirmed |
| Start | Begin work | Only after confirmation |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Asking all questions at once | One at a time. Always. |
| Starting before confirmation | Get the explicit "yes" first. |
| Restating incorrectly and not catching it | Make your restatement specific enough the user can spot errors. |
| Skipping because "it matches a pattern you've seen" | Patterns have variations. Confirm this specific case. |
| Treating confirmation as optional | Silence is not confirmation. Ask explicitly. |
| Proposing an alternative solution instead of clarifying the requested one | Understand what the user wants first. Suggest alternatives only after. |
| Assuming location/ownership | Where something lives (global vs project, which repo) is a requirement. Ask. |
