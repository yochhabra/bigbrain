---
name: bigbrain-pr
description: "Auto-generate PR descriptions from task context, diffs, and Jira tickets. Use when user says 'generate PR description', 'write PR summary', 'describe this PR', or when BigBrain creates a PR after task completion."
---

# BigBrain PR — Auto PR Description Generator

## When to Generate

1. **After BigBrain task completes:** Worker Claude pushed, BigBrain approved → generate PR description
2. **Manual:** User says "generate PR description for branch X"
3. **On PR creation:** When BigBrain or user opens a PR via `gh pr create`

## Data Sources

1. **BigBrain task YAML** — description, jira link, compass_milestone
2. **Task result JSON** — summary, files_changed, decisions_made
3. **Git diff** — `mcp__toolshed__code_get_branch_diff` for the branch vs base
4. **Jira ticket** — `mcp__toolshed__get_jira_ticket` for context, acceptance criteria
5. **Commit messages** — from the branch's commits

## Output Format (Stripe PR template)

```markdown
## Summary

{1-2 sentences: what this PR does and why}

## Jira

{JIRA_KEY}: {ticket title}

## What Changed

- {bullet per logical change, grouped by concern}

## Design Decisions

- {non-obvious choices from task's `decisions_made` field}

## Testing

- {what was tested: unit tests, QA verification, local dev}
- {test commands run}

## Risks

- {anything reviewers should watch for}
- {rollback plan if applicable}
```

## Flow

1. Identify the branch (from task YAML, or user specifies)
2. Get the diff (`code_get_branch_diff` with base branch from project stack)
3. Get Jira context if linked
4. Read task result for summary and decisions
5. Generate description
6. Present to user for review
7. If approved: either output for copy-paste, or pass to `gh pr create --body`

## Rules

- Keep Summary to 1-2 sentences — reviewers skim
- Design Decisions section only if non-obvious choices were made
- Always include Jira link if available
- Never fabricate testing details — only include what was actually tested
- If the diff is large (> 500 lines), group changes by file/concern with sub-headers
