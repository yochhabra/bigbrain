---
name: bigbrain-ci
description: "Monitor CI and auto-fix failures. Use when user says 'watch CI', 'CI failed on branch X', 'fix CI', 'check CI', or triggered automatically after a push via bigbrain-ci-poll."
---

# BigBrain CI — Auto-Fix Loop

## Triggers

1. **After push:** BigBrain adds branch to `~/.claude/bigbrain/ci-watch.txt`, starts polling
2. **Manual:** User says "CI failed on branch X" or "fix CI on X"
3. **Scheduled wakeup:** `bigbrain ci-poll <branch> <devbox>` from ScheduleWakeup

## Event-Triggered Polling

No idle polling. CI checks only happen after a push.

When BigBrain collects a task result with `"pushed": true`:
1. Add branch to `~/.claude/bigbrain/ci-watch.txt`
2. Immediately check `mcp__toolshed__ci_v2_status` for repo + branch
3. If still running → `ScheduleWakeup` for 2 min: `"bigbrain ci-poll <branch> <devbox>"`
4. On wakeup → check again. Max 10 retries (20 min).
5. If passed → remove from watch list, done
6. If failed → enter fix loop
7. If max retries → stop, note CI status unknown in task

**Cost:** Near-zero. Each poll is one MCP tool call, not a Claude invocation.

## Diagnosis

1. Get build token from CI status response
2. Get failures: `mcp__toolshed__ci_v2_failures` with build token
3. Classify:
   - `autofix` command present → auto-fixable
   - `infra_flake: true` on all failures → transient, re-push
   - `scripts` array non-empty → run scripts
   - Otherwise → real test failure

## Auto-Fix Flow

**Auto-fixable (autofix command):**
```
"CI failed with auto-fixable issues. Run from /pay/src/pay-server:
{autofix_command}
Then commit and `pay stack push`. Report result."
```
Max 2 retries, then escalate.

**Script-fixable (scripts array):**
```
"CI failed. Run these scripts to regenerate files:
{scripts list}
Then commit and push. Report result."
```

**Infra flakes:**
- Don't fix. Tell user: "CI had infra flakes, likely transient."
- Optionally re-push (with confirmation).

**Real test failures:**
1. Get details: `mcp__toolshed__ci_v2_failure_details` for representative UIDs
2. Assign fix task with failure context:
   ```
   "CI failed with test failures:
   Signatures: {signatures}
   Representative: {test_uids with messages}
   Investigate, fix, run locally, push."
   ```
3. If blocked/fails twice → escalate

## Escalation

```
CI FAILURE on {branch}:
Build: {build_token}
Failures: {count} ({unique_signatures} unique)
Top signature: {signature}
Auto-fix attempted: {yes/no}
Result: {still failing}
Options: A) Another fix attempt  B) You investigate
Representative test: {test_uid}
```

## Integration

After task approved + pushed:
1. Poll CI
2. If fails → auto-fix loop
3. If passes → truly done, update Jira
