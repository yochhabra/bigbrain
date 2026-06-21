---
name: bigbrain-code-review
description: "Dedicated code review agent that reviews worker Claude output before human review. Use when BigBrain completes a task review cycle, or user says 'review the code', 'code review on branch X', 'get a second opinion'."
---

# BigBrain Code Review — Independent Review Agent

## Purpose

A separate review pass that acts as a senior reviewer BEFORE code reaches the human. The worker Claude wrote and tested the code — this agent only reviews, never modifies.

The review agent uses a different perspective: it hasn't seen the implementation process, only the final diff. This catches issues the implementer is blind to.

## When to Run

1. **Automatic:** After BigBrain approves a task's functionality (it works, CI passes), run code review before marking truly done
2. **Manual:** User says "review the code on branch X" or "get a second opinion"

## Review Flow

### Input

1. Get the branch diff: `mcp__toolshed__code_get_branch_diff` (repo, base_branch, branch)
2. Read the task description from `~/.claude/bigbrain/tasks/<task-id>.yaml`
3. Read the Jira ticket if linked (for acceptance criteria)
4. Read relevant existing code in the same package (for pattern matching)

### Review Dimensions

The review agent evaluates across these dimensions, scored 🟢/🟡/🔴:

**1. Correctness**
- Does the code do what the task/Jira asked for?
- Edge cases handled? Null safety?
- Error handling appropriate?

**2. Design Quality**
- Single responsibility per class/method?
- Methods < 15 lines? Classes < 200 lines?
- Right pattern used? (strategy vs if/else, builder vs constructor sprawl)
- Composition over inheritance?

**3. Codebase Consistency**
- Matches existing patterns in the same package?
- Naming follows conventions?
- File organization matches neighbors?

**4. Readability**
- Can you understand what a method does from its name alone?
- No unnecessary complexity?
- No dead code or commented-out code?

**5. Testing**
- Are the new paths tested?
- Are edge cases covered?
- Test names describe behavior, not implementation?

**6. Security & Safety**
- No hardcoded secrets or credentials?
- Input validation at boundaries?
- No injection vectors?

### Output Format

```markdown
## Code Review: {branch}

**Overall:** 🟢 Approve / 🟡 Approve with suggestions / 🔴 Request changes

### Scores
| Dimension | Score | Notes |
|-----------|-------|-------|
| Correctness | 🟢 | |
| Design | 🟡 | ApprovalHandler is doing too much |
| Consistency | 🟢 | Matches existing notification patterns |
| Readability | 🟢 | Clear method names |
| Testing | 🟡 | Missing edge case for expired approvals |
| Security | 🟢 | |

### Must Fix (🔴 blocking)
- {critical issues that must be resolved}

### Should Fix (🟡 suggestions)
- {improvements that make the code better but aren't blocking}
- {include file:line references}

### Nits (optional)
- {stylistic preferences, take-or-leave}
```

## Integration with BigBrain

### In the task lifecycle:

```
Worker Claude implements → BigBrain checks (works? CI passes?) 
  → Code Review Agent reviews (quality? design? patterns?)
    → If 🟢: mark done, generate PR description, notify human "PR ready for your review"
    → If 🟡: mark done but include suggestions in PR description
    → If 🔴: send feedback to worker Claude for fixes (resume session)
```

### Re-assignment on 🔴:

When code review returns "request changes":
1. Format the review findings as a follow-up task
2. Resume the worker Claude's session with:
   ```
   "Code review found issues. Fix these before we can ship:
   {must_fix items with file:line references}
   
   Then push again."
   ```
3. After fix, run code review again (max 2 cycles, then escalate)

## Review Agent: Codex (OpenAI)

The review uses a COMPLETELY DIFFERENT AI provider — OpenAI's Codex — for genuinely independent perspective. The worker Claude implemented the code; Codex reviews it without sharing any context or biases.

### Invocation

```bash
ssh -o ControlPath=/tmp/bigbrain-ssh-%h <devbox_host> "cd /pay/src && codex review --base <base_branch> '<custom review instructions>'"
```

**Custom review instructions** (passed as prompt):
```
Review this diff for:
1. Correctness — does it handle edge cases and nulls?
2. Design — single responsibility, short methods (<15 lines), right patterns?
3. Consistency — matches existing code in the same package?
4. Readability — clear names, no unnecessary complexity?
5. Testing — are new paths covered?
6. Security — no hardcoded secrets, input validation at boundaries?

Flag blocking issues as MUST FIX. Flag improvements as SHOULD FIX. Be specific with file:line references.
```

### Available Review Tools on Devbox

| Tool | Command | Model | Best for |
|------|---------|-------|----------|
| **Codex** (primary) | `codex review --base <branch>` | OpenAI o3/o4 | Deep reasoning, catches logic errors |
| **Goose** (alternate) | `pay goose session start --prompt "review..."` | Varies | Stripe-specific patterns, cursor rules |
| **Claude** (fallback) | `claude -p --model sonnet "review..."` | Sonnet | Fast, checklist-oriented |

**Default: Codex.** Use Goose as alternate if Codex is unavailable. Claude Sonnet as last resort (but same provider as worker, less independent).

### Parsing Codex Output

Codex review outputs markdown. BigBrain parses it to determine:
- If "MUST FIX" items present → treat as 🔴 request changes
- If only "SHOULD FIX" → treat as 🟡 approve with suggestions
- If neither → treat as 🟢 approve

## Important Rules

- The review agent NEVER modifies code — only evaluates
- The review agent has NOT seen the implementation process — fresh eyes
- Don't block on nits — only 🔴 for real issues
- If the worker Claude already made a documented decision (in `decisions_made`), don't re-litigate it unless it's clearly wrong
- Include file:line references so feedback is actionable
