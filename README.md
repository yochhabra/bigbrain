# BigBrain

Local-only Claude Code orchestration system. Runs on your machine, coordinates autonomous Claude agents across Stripe devboxes.

**This repo is NOT deployed anywhere.** It's version control for local skills and templates only.

## Structure

```
skills/
  bigbrain/SKILL.md     # Orchestration skill (symlinked to ~/.claude/skills/bigbrain/)
templates/
  task-context.md       # Template for task context files
  task.yaml             # Template for task definitions
  project.yaml          # Template for project registry
```

## Setup

```bash
git clone https://github.com/yochhabra/bigbrain.git ~/bigbrain
ln -sf ~/bigbrain/skills/bigbrain ~/.claude/skills/bigbrain
```

## Runtime State (NOT in this repo)

State lives in `~/.claude/bigbrain/` (ephemeral, not version-controlled):
- `projects/` — active project registry
- `tasks/` — task queue
- `decisions/` — decision audit log
- `contexts/` — generated task context files
- `snippets/` — weekly snippet history
- `compass/` — compass update history
