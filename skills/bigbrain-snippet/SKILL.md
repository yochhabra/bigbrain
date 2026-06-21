---
name: bigbrain-snippet
description: "Generate weekly snippet for go/snippets. Use when user says 'generate snippet', 'weekly update', 'write my snippet', or it's Tuesday and snippet is in today.md."
---

# BigBrain Snippet — Weekly Update Generator

## Data Sources

Gather completed work from ALL sources since last Tuesday:

1. **Obsidian** — scan `today.md` history files, extract `🟢` rows
2. **BigBrain tasks** — `~/.claude/bigbrain/tasks/*.yaml` with `status: done` since last Tuesday
3. **Pull Requests** — `mcp__toolshed__code_search_pull_requests` for author `yochhabra`
4. **Jira** — `mcp__toolshed__search_jira`: `assignee = yochhabra AND updated >= "-7d"`

## Deduplication

- Match by Jira ticket key across sources
- Match by branch name (PR branch often contains ticket key)
- Present each piece of work once, with all artifacts

## Output Format

```
## What I shipped
- {completed items with PR/Jira links}

## In progress
- {items started but not done}

## Reviews & collaboration
- {PRs reviewed, pairing sessions, design discussions}

## Planning & docs
- {docs written, plans made, meetings that produced decisions}
```

## Snippet History

**Location:** `~/.claude/bigbrain/snippets/YYYY-MM-DD.md`

**Before generating:**
- Read last 2-3 snippets for style + avoid repeats
- Track "in progress" → "shipped" transitions

**After approval:**
- Save to snippets history
- Prune files > 2 months: `tasks.sh prune-snippets`

## Reconciliation

After presenting draft: "Anything missing? (meetings, ad-hoc help, discussions without artifacts)"
Incorporate additions, then save.
