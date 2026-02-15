# RUNTIME EXECUTION MODEL v1.0

## HOW KAI ACTUALLY RUNS

Kai is not a persistent process. Kai is an orchestrated system with three execution modes:

### Mode 1: n8n Webhook-Triggered (Primary — Phase 0)

External events (new lead, CI failure, new review) fire webhooks into n8n workflows.
Each n8n workflow:
1. Receives the webhook payload
2. Normalizes the data
3. Calls Claude API with the system prompt + relevant context files
4. Routes the LLM response (Telegram ping, draft artifact, log entry)
5. Logs the intervention to `proactive/interventions.jsonl`

**No persistent agent state between webhook invocations.** Each invocation is stateless from the LLM's perspective. State lives in the filesystem (beliefs, episodes, cost ledger) and is injected into context as needed.

### Mode 2: Heartbeat Cron (Phase 0+)

`heartbeat.mjs` runs on a cron schedule (via systemd, pm2, or n8n cron trigger).
On each tick:
1. Check kill switch file existence
2. Check budget ceilings (hourly/daily)
3. Evaluate each cron-triggered trigger in `proactive/triggers.yaml`
4. For triggers whose conditions are met: assemble context, call Claude API, route response
5. Log all actions to `audit.jsonl`

### Mode 3: Interactive Chat (Clawdbot — Phase 0+)

When Shuki opens a Clawdbot chat session:
1. `SYSTEM_PROMPT.md` is injected as the system message
2. Referenced files (POLICY, SOUL, MEMORY, etc.) are concatenated into the system context or fetched via tool calls, depending on context window budget
3. The session is stateful for the duration of the conversation
4. On session end, any new beliefs or lessons are persisted to the filesystem

## CONTEXT ASSEMBLY

The boot sequence in `SYSTEM_PROMPT.md` defines file loading order. In practice:

**For n8n workflows**: Only the files relevant to that trigger are included. The lead SLA guardian includes POLICY summary + lead-specific beliefs + cost gate. It does NOT load INTELLECT.md, EVOLVE.md, or reading_queue.json.

**For heartbeat cron**: The heartbeat script reads triggers.yaml, checks conditions, and only assembles context for triggered actions.

**For Clawdbot chat**: Full boot sequence is loaded. Phase 3 files (INTELLECT, EVOLVE, AGENTS) are excluded until those phases are active, saving ~3000 tokens per session.

## STATE PERSISTENCE

| State Type | Storage | Access Pattern |
|-----------|---------|---------------|
| Beliefs | `memory/beliefs.jsonl` (append-only) + `memory/beliefs.snapshot.json` (materialized) | Snapshot for fast reads, JSONL for audit |
| Episodes | `memory/episodes.jsonl` | Append-only |
| Cost ledger | `memory/cost_ledger.jsonl` | Append-only, summed by heartbeat |
| Interventions | `proactive/interventions.jsonl` | Append-only, used for trust ramp |
| Audit trail | `.openclaw/workspace/audit.jsonl` | Append-only, never modified |
| Checkpoints | `runs/<run_id>/state.json` | Written per-step, read on crash recovery |

## INTEGRITY CHECKS (Orchestration Layer Responsibility)

These checks run in code (heartbeat.mjs / n8n), NOT in the LLM:

1. **Governance hash**: SHA-256 of `schemas/governance.json` computed on first run, verified on every subsequent startup
2. **Kill switch**: Check for `.openclaw/KILL` file before every action
3. **Budget enforcement**: Sum cost_ledger.jsonl before every LLM call
4. **Audit immutability**: audit.jsonl is opened append-only; any truncation detected = halt

## PHASE 0 INTEGRATION STATUS

| Integration | Status | Channel |
|---|---|---|
| Telegram Bot | Active | n8n + Clawdbot |
| Lead webhook (n8n) | Active | n8n |
| Claude API | Active | Both |
| GitHub API | Planned (Phase 1) | n8n |
| RepairDesk API | Planned (Phase 1) | n8n |
| Google Calendar | Planned (Phase 1) | n8n |
| HubSpot | Not decided — leads may come from RepairDesk or website form | n8n |
| ReviewGuard webhook | Planned (Phase 1) | n8n |

## FILESYSTEM PATHS

This project runs on Windows (Shuki's environment). Paths use the workspace root:

```
OPENCLAW_WORKSPACE = C:\Users\shuki\Downloads\Clowdbot  (or set via env var)
Kill switch: %OPENCLAW_WORKSPACE%\.openclaw\KILL
Audit log:   %OPENCLAW_WORKSPACE%\.openclaw\workspace\audit.jsonl
Locks:       %OPENCLAW_WORKSPACE%\.openclaw\locks\
```

The heartbeat script and n8n workflows resolve paths relative to `OPENCLAW_WORKSPACE`. Unix-style paths in documentation (e.g., `~/.openclaw/KILL`) are shorthand for the workspace-relative equivalent.
