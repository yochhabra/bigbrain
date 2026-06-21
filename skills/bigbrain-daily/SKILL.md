---
name: bigbrain-daily
description: "Generate today's focus and prioritize tasks. Use when user says 'what's next', 'plan my day', 'what should I do today', 'today's tasks', or 'prioritize'."
---

# BigBrain Daily — Task Prioritization & today.md

## Obsidian File Structure

**Base path:** `/Users/yochhabra/Documents/Obsidian/Approvals/Tasks/`

```
Tasks/
├── today.md              # 3-5 deep focus items + ⚡/⏳ items
├── backlog.md            # Prioritized near-term queue
├── long-term/
│   ├── techdebt.md
│   ├── learning.md
│   └── improvements.md
└── <Month>/Week <range>/<DD Month>.md   # Historical daily logs
```

**Table format:**
```markdown
| # | Task | Context | Status |
|---|------|---------|--------|
| 1 | Task description | links, notes | ⚪ |
| 1.1 | ↳ sub-task | | ⚪ |
```

**Status:** `⚪` Todo | `🟡` In Progress | `🟢` Done | `🔴` Blocked
**Tags:** `⚡` quick (<15 min) | `⏳` waiting on others | `🚨` deadline alert

## Generating today.md

1. **Archive yesterday:** Copy `today.md` to `<Month>/Week <range>/<DD Month>.md`
2. **Clean backlog:** Remove `🟢` rows from `backlog.md` (use `tasks.sh clean-done`)
3. **Select today's focus:**

### Priority order (fill slots top-down):
1. Unblocking others (PR reviews, doc reviews) — always first
2. Items blocking devbox Claudes (`needs_decision`) — they're waiting
3. Compass milestone critical path — nearest deadline first
4. Deadline-driven items — by proximity
5. If slots remain: rotate one item from `long-term/`

### Flexible item limit:
- Max 5 DEEP FOCUS tasks (sustained concentration)
- Quick tasks (`⚡`) and waiting tasks (`⏳`) don't count toward limit
- Realistic today.md: 3-4 deep tasks + several ⚡/⏳

### Milestone proximity alerts:
- Milestone < 1 week away + unplanned work → add `🚨 Plan: {task}` to today
- Milestone has unknowns/risks → add `De-risk: {unknown}` to today
- Show days remaining in Context column

### Long-term rotation:
- Only if Compass load is light (< 4 critical items)
- Rotate: techdebt → learning → improvements → techdebt...
- Pick items relating to current project when possible
- Max 1 long-term item per day

### Recurring tasks (non-negotiable):
- **Monday:** "BigBrain weekly review"
- **Tuesday:** "Write weekly snippet" + "Write Compass update"
- **Milestone target dates:** "Compass update — {milestone}"
- **Status change to at_risk:** "Compass update — {project} now at risk"

## Proactive Task Creation

BigBrain identifies gaps and creates planning tasks:

1. Milestone < 2 weeks, remaining_tasks have no corresponding BigBrain task → add to backlog: `Plan & break down: {task}`
2. Milestone < 1 week, items unplanned → add to today.md
3. Milestone turned at_risk → add to today: `Reassess {milestone}`
4. Worker Claude `follow_up_needed` → add to backlog
5. New Jira tickets assigned to you → add to backlog

## Compass Milestones Cache

**Location:** `~/.claude/bigbrain/compass/milestones/<project-slug>.yaml`

Daily refresh (once per day, first invocation):
1. For each active project with `compass_url` → fetch via `mcp__toolshed__execute_internal_search`
2. Update milestones YAML with current status, dates, remaining tasks
3. Skip if `last_fetched` < 24 hours ago

## Adding Tasks

- **Urgent:** Directly to `today.md` (if < 5 deep items)
- **Normal:** Append to `backlog.md`
- **Long-term:** Appropriate `long-term/*.md` file

Rules: only append, never modify existing rows, never change status.

## Cross-referencing

When BigBrain assigns a task linked to an Obsidian item:
- Context column: `🤖 task-003 on devbox:add-approve-button`
- On complete: `🤖 task-003 ✓ — PR ready for review`
- Never change status to 🟢 — only user does that.

## Shell Utilities

`~/.claude/bigbrain/scripts/tasks.sh` — archive, clean-done, renumber, add-backlog, add-today, set-status, set-context
