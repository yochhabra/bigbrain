# Task: {title}

## Project Context
{high-level what this project is about, why it exists}

## Jira Ticket: {jira_key}
{Jira description, acceptance criteria, and relevant comments — fetched by BigBrain}

## Your Assignment
{specific task description — what to implement, where}

## Constraints
- Repo: {repo}
- Workspace: {workspace}
- Branch: {branch}
- Base: {base branch}
- Must pass: {test/lint commands}
- Follow all rules in ~/.claude/rules/

## Prior Work
{what's already done on this branch, relevant decisions from earlier tasks}

## Definition of Done
- [ ] Implementation complete
- [ ] Tests pass
- [ ] Lint passes
- [ ] Pushed to branch via `pay stack push`
- [ ] Output structured JSON per ~/.claude/rules/status-reporting.md

## Important
- For routine decisions (naming, structure): decide and document in `decisions_made`
- For major decisions (architecture, APIs, dependencies): report as "blocked" with `needs_decision`
- You are NOT the final decision-maker on non-trivial choices
