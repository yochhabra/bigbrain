# BigBrain Project

This repo contains the BigBrain orchestration skill and templates. It runs ONLY on the user's local machine — never on devboxes.

## Repo structure

- `skills/bigbrain/SKILL.md` — The main orchestration skill definition. Symlinked to `~/.claude/skills/bigbrain/`
- `templates/` — YAML/MD templates for projects, tasks, and contexts

## Development guidelines

- The SKILL.md is the source of truth for BigBrain's behavior
- When editing SKILL.md, also update the local symlink (`~/.claude/skills/bigbrain/SKILL.md`) so changes take effect immediately
- Templates define the schema — keep them in sync with what SKILL.md references
- Runtime state (projects, tasks, snippets, compass) lives in `~/.claude/bigbrain/`, NOT in this repo

## Testing changes

After editing SKILL.md, test by:
1. Starting a new Claude session
2. Saying "bigbrain status" to verify the skill loads
3. Running a simple task assignment against a test devbox

## Coordination with devbox-config

This repo defines what BigBrain does. The [devbox-config](https://github.com/yochhabra/devbox-config) repo defines what worker Claudes do. Changes to the output schema (status-reporting.md) or work rules (autonomous-work.md) go in devbox-config, not here.
