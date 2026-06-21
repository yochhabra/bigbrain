# BigBrain

A multi-devbox orchestration system that acts as a **team lead** for autonomous Claude Code agents across Stripe devboxes.

## What it does

BigBrain runs on your local machine and coordinates work across multiple Stripe devboxes. It:

- **Assigns tasks** to devbox Claudes with full context (Jira tickets, project state, prior work)
- **Monitors progress** — gets notified when tasks complete, parses structured results
- **Reviews output** — checks diffs, CI status, validates against task requirements
- **Makes routine decisions** — approves straightforward work, re-assigns with feedback
- **Escalates** — brings context TO you when decisions need human judgment (no context-switching needed)
- **Manages stacks** — tracks stacked PRs, coordinates restacks when parent branches merge
- **Spawns/stops devboxes** — ephemeral, single-use devboxes per task (with your confirmation)
- **Generates reports** — weekly snippets (go/snippets) and Compass updates from all data sources
- **Manages your task list** — adds items to your Obsidian daily task file

## Architecture

```
Your Machine (BigBrain)
├── ~/.claude/skills/bigbrain/     # This skill (symlinked from this repo)
├── ~/.claude/bigbrain/            # Runtime state (not in this repo)
│   ├── projects/                  # Active project registry
│   ├── tasks/                     # Task queue with status
│   ├── decisions/                 # Decision audit log
│   ├── contexts/                  # Task context files sent to devboxes
│   ├── snippets/                  # Weekly snippet history
│   └── compass/                   # Compass update history
├── SSH ControlMaster tunnels      # Persistent connections to devboxes
└── Background tasks               # Parallel claude -p invocations

Devboxes (provisioned via github.com/yochhabra/devbox-config)
├── ~/.claude/rules/autonomous-work.md    # How to work unsupervised
├── ~/.claude/rules/status-reporting.md   # Structured JSON output schema
├── ~/.claude/settings.local.json         # Locked-down permissions
└── ~/.claude/current-task.md             # Written by BigBrain per task
```

## How it works

1. You say "run bigbrain" or "assign tasks"
2. BigBrain reads pending tasks, picks the highest priority based on Compass milestones
3. Fetches Jira context, generates task file, SCPs it to the devbox
4. Invokes `claude -p --output-format json` via SSH (background, ControlMaster for speed)
5. Devbox Claude works autonomously: implements, tests, pushes via `pay stack push`
6. BigBrain gets notified on completion, parses structured JSON result
7. Reviews the diff and CI status
8. Approves, requests changes, or escalates to you

## Commands

| Command | What it does |
|---------|-------------|
| `run bigbrain` / `assign tasks` | Assign next pending task to a devbox |
| `bigbrain status` | Report all active projects and tasks |
| `add project` | Create a new project in the registry |
| `add task` | Create a task for an existing project |
| `review` | Check completed tasks and run review flow |
| `what's next` | Prioritized recommendation based on Compass milestones |
| `generate snippet` | Draft weekly snippet from all sources |
| `compass update` | Draft Compass project update for leadership |

## Key design decisions

- **SSH ControlMaster** — 16s → 1.2s connection reuse per devbox
- **`--resume` sessions** — devbox Claude retains context across tasks on same project
- **Structured JSON output** — parseable results with status, blockers, decisions
- **Branch safety** — worker Claudes cannot checkout/create branches, only `pay stack push`
- **Ephemeral devboxes** — created per task, stopped when PR merges, never reused
- **Low escalation threshold** — BigBrain asks you freely, only handles truly routine decisions

## Setup

```bash
# Clone this repo
git clone https://github.com/yochhabra/bigbrain.git ~/bigbrain

# Symlink skill into Claude Code
ln -sf ~/bigbrain/skills/bigbrain ~/.claude/skills/bigbrain

# Create runtime state directories
mkdir -p ~/.claude/bigbrain/{projects,tasks,decisions,contexts,snippets,compass}
```

## Related repos

- [devbox-config](https://github.com/yochhabra/devbox-config) — Rules and settings deployed to devboxes (the "employee handbook" for worker Claudes)
- `pay-server/devbox/dotfiles/yochhabra/` — Bootstrap script that clones devbox-config on devbox creation
