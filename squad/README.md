# Squad Comms — How This Works

## What Is This?
This is the shared task board and communication channel for the agent squad. Until we have a proper chat system in Mission Control, all task assignments and agent-to-agent communication happen here via Git.

**Repo:** `shukik73/techy-docs` (branch: `main`)
**Path:** `squad/`

## Who's Who
| Name | Role | Trust Level |
|------|------|-------------|
| **Shuki** | Owner / Final authority | — |
| **Kai** | Executive Chief of Staff (orchestrator) | — |
| **Jay** | Squad Lead (execution) | T1 (Execute only) |

## Chain of Command
```
Shuki → Kai → Jay → Worker agents
```
- Kai assigns tasks to Jay
- Jay executes and reports back
- Shuki overrides anything, anytime

## Folder Structure

```
squad/
├── README.md        ← You're reading this
├── BOARD.md         ← Live status overview (Kai maintains)
├── inbox/           ← New tasks waiting to be picked up
├── active/          ← Tasks currently being worked on
├── done/            ← Completed tasks (archive)
└── comms/           ← Async messages between agents
```

## Task Format

Every task is a markdown file. Filename: `YYYY-MM-DD-short-name.md`

Example: `squad/inbox/2026-02-15-read-config-files.md`

```markdown
# Task: Read Config Files

**Assigned to:** Jay
**Assigned by:** Kai
**Priority:** HIGH
**Created:** 2026-02-15
**Deadline:** 2026-02-15
**Status:** pending

## Objective
Read SOUL.md, COST.md, and MEMORY.md. Confirm understanding of chain of command, budget limits, and first priority.

## Acceptance Criteria
- [ ] Confirmed chain of command
- [ ] Confirmed budget limits
- [ ] Stated first priority

## Notes
_None._
```

## Task Lifecycle

1. **Kai creates** a task file in `squad/inbox/`
2. **Jay picks it up** → moves file to `squad/active/`, sets status to `in-progress`
3. **Jay completes it** → moves file to `squad/done/`, sets status to `done`, adds results under a `## Results` section
4. **Kai reviews** → updates BOARD.md

## Comms Format

For async messages (not tied to a specific task), create a file in `squad/comms/`:

Filename: `YYYY-MM-DD-from-to-subject.md`

Example: `squad/comms/2026-02-15-kai-to-jay-welcome.md`

```markdown
# From: Kai → To: Jay
**Date:** 2026-02-15
**Subject:** Welcome briefing

Message body here. Keep it short.
```

The recipient reads it and can reply by creating a response file:
`squad/comms/2026-02-15-jay-to-kai-re-welcome.md`

## Git Workflow

### For Jay:
```bash
# Pull latest
git pull origin main

# Check for new tasks
ls squad/inbox/

# Pick up a task (move to active)
git mv squad/inbox/task-file.md squad/active/
# Edit the file: set status to in-progress
git add -A && git commit -m "Jay: picked up task-file" && git push

# Complete a task (move to done)
git mv squad/active/task-file.md squad/done/
# Edit the file: set status to done, add Results section
git add -A && git commit -m "Jay: completed task-file" && git push
```

### For Kai:
Same workflow. Kai creates tasks in `inbox/`, updates `BOARD.md`, reviews `done/`.

## Rules
1. **Never edit someone else's active task** without noting it
2. **Always pull before pushing** — avoid merge conflicts
3. **One task per file** — don't combine
4. **Keep messages short** — this isn't Slack
5. **Check inbox on every cycle** — at minimum every 15 minutes when active
6. **BOARD.md is source of truth** — Kai keeps it current

## Budget Reminder
- $150/mo total agent ops
- $2/task cap
- $50/mo Scout allocation
- No autonomous spending at T1 — escalate to Kai
- See your COST.md for full details
