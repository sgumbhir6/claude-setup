---
name: sync-skills
description: Sync skills from ~/.claude/skills (source of truth) to ~/eater-real-time-pricing/sgumbhir/.claude/skills. One-way copy — global to repo.
autoInvoke: false
---

# Sync Skills

One-way sync of a specific skill from global to the repo copy:
- **Source**: `~/.claude/skills/<skill-name>/`
- **Target**: `~/eater-real-time-pricing/sgumbhir/.claude/skills/<skill-name>/`

## Procedure

1. Ask the user which skill to sync. The skill name should match a directory under `~/.claude/skills/`.

2. Run rsync for that specific skill:

```bash
rsync -av ~/.claude/skills/<skill-name>/ ~/eater-real-time-pricing/sgumbhir/.claude/skills/<skill-name>/
```

3. Report what was synced. If nothing transferred, report "Skill is already in sync."
