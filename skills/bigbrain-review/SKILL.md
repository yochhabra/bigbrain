---
name: bigbrain-review
description: "Monday weekly review — costs, outcomes, optimizations, and claude.md improvement suggestions. Use when user says 'weekly review', 'Monday review', 'how did this week go', or it's Monday."
---

# BigBrain Review — Weekly Analysis

## Gather

1. All tasks from `~/.claude/bigbrain/tasks/` completed in past 7 days
2. Cost per task (`cost_usd` field)
3. Duration per task (`duration_minutes`)
4. Retry counts (re-assignments)
5. Escalations (times BigBrain asked human)
6. Feedback entries from `~/.claude/bigbrain/feedback/`

## Report Format

```
## BigBrain Weekly Review — Week of {date}

### Summary
- Tasks completed by devbox Claudes: {N}
- Tasks completed by human: {N}
- Total cost: ${X.XX}
- Avg cost per task: ${X.XX}
- Avg duration per task: {N} min
- Escalations to human: {N}
- Auto CI fixes: {N} successful, {N} failed

### Cost Breakdown
| Task | Project | Duration | Cost | Retries |
...

### What Went Well
- {tasks that completed first try, no escalation}

### What Needed Intervention
- {tasks that failed, got stuck, or escalated}

### Optimization Suggestions
- {patterns from failures}
- {cost outliers — vague prompts causing exploration}
- {long-term items that relate to this week's pain}
```

## Claude.md / Rules Improvement Suggestions

Analyze this week's failures, retries, escalations, and feedback for patterns:

- Same mistake 2+ times → needs a rule
- Wrong design choice → add to `design-quality.md`
- Didn't know a tool/command → add to CLAUDE.md
- Misunderstood codebase convention → add to project CLAUDE.md

**Output:**
```
### Suggested Rule Updates

1. **Add to design-quality.md:** "{rule}"
   Reason: {what went wrong, frequency}

2. **Add to CLAUDE.md:** "{guidance}"
   Reason: {what was missing}

3. **Add to autonomous-work.md:** "{behavior change}"
   Reason: {escalation that could have been avoided}
```

After user approves: apply changes, push to devbox-config repo.
