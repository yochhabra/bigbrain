---
name: bigbrain-compass
description: "Generate Compass project updates for leadership. Use when user says 'compass update', 'update compass', or it's Tuesday/milestone target date."
---

# BigBrain Compass — Project Update Generator

## Key Differences from Snippets

- **Audience:** Leadership (manager's manager)
- **Scope:** One specific project/milestone
- **Tone:** BLUF-first, outcome-focused, blameless
- **Cadence:** Tuesdays + target dates + status changes

## Data Sources

1. **Project YAML** — `~/.claude/bigbrain/projects/`
2. **Completed tasks** — for this project, this week
3. **Jira** — milestone progress, ticket status
4. **PRs** — merged PRs for project branches
5. **Past updates** — `~/.claude/bigbrain/compass/<project-slug>/`
6. **Obsidian** — completed items for this project

## Output Format (DevProd template)

```
**Status:** 🟢 On track / 🟡 At risk / 🔴 Off track

**BLUF:** {1-2 sentences. What functionality was unlocked? Progressing as expected?}

**Progress this week:**
- {outcome-focused bullets — what shipped/unblocked}

**Next steps:**
- {next week's plan}

**Risks/blockers:**
- {blameless tone}
```

## Guidelines (from go/compass docs)

- BLUF first — leadership reads to know if you need support
- Blameless — "we identified additional requirements" not "team X didn't deliver"
- Outcome-focused — functionality unlocked, not activity performed
- Don't fear yellow/red — they get you support
- At risk = plan exists but needs things to happen
- Off track = no clear path back to green yet

## Compass History

**Location:** `~/.claude/bigbrain/compass/<project-slug>/YYYY-MM-DD.md`

Before generating:
- Read last 2-3 updates for continuity
- Check "next steps" from last week → should appear in "progress" if done
- Track status trajectory (green → yellow needs explanation)

After approval: save + prune > 2 months.

## Flow

1. Ask which project (if multiple active)
2. Gather data from all sources
3. Determine status (green/yellow/red) from milestone progress vs. deadlines
4. Draft update
5. "Here's your Compass update for {project}. Status: {color}. Adjust anything?"
6. Save approved version
