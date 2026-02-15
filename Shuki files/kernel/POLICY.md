# KERNEL GOVERNANCE POLICY (v1.0)

**THIS FILE IS IMMUTABLE. No agent may modify it. Changes require manual user edit only.**

## 1. ACCESS CONTROL

- **Privilege Level**: All sub-agents operate in stateless containers (when multi-agent mode is active in Phase 2+).
- **Restricted Paths**: No agent may write to `/kernel`, access `.ssh`, or modify `SOUL.md` governance sections.
- **Tool Veto (Phase 2+)**: CEO agent is barred from `bash` and `edit` tools; it must delegate to Worker agents. See Section 1a for Phase 0 override.

### 1a. PHASE 0 TOOL OVERRIDE (Single-Agent Mode)

During Phase 0, no Worker agents exist. The single agent operates with expanded permissions under these constraints:

- **Allowed**: Read, Write, Edit, Bash (non-destructive), Plan, Search, TodoWrite
- **Forbidden**: Bash (destructive: `rm -rf`, `drop table`, production deployments), Spawn, modify governance files
- **HITL Required**: File deletion, credential updates, production deployment, external communication to new domains
- **Audit**: ALL tool calls logged to `audit.jsonl` regardless of risk level
- **Graduation**: Phase 0 tool override expires when multi-agent mode (Phase 2) is activated. At that point, standard CEO/Worker boundaries in Section 1 apply.

## 2. AUTONOMY & TRUST RAMPS

- **Default State**: L1 (Suggest Only). No external communication (email/SMS) without L2+ approval.
- **High-Risk Trigger**: File deletion, credential update, or production deployment triggers HITL approval via Telegram pairing code.
- **Promotion**: Earned through accepted interventions, tracked in `proactive/interventions.jsonl`.
- **Demotion**: Immediate on "annoyed/stop" feedback or high-risk mistake.

## 3. FINANCIAL GOVERNANCE

- **API Budget**: Max $0.50/hour; $5.00/day across all models.
- **Usage Tracking**: Every API call logged to `memory/cost_ledger.jsonl` with timestamp, reason, and token count.
- **Decision Gate**: Proactive actions only execute if `ExpectedFutureCost > InterventionCost x (1 + AnnoyanceRisk)`. See `docs/COST.md` for Phase 0 lookup table.
- **Enforcement**: Budget ceilings are enforced by the orchestration layer (heartbeat.mjs / n8n), not by the LLM.

## 4. AUDIT & LOGGING

- **Immutability**: `audit.jsonl` is append-only. Any agent attempt to delete or modify audit logs triggers immediate session freeze.
- **Retention**: 90 days minimum.
- **Contents**: All tool calls, all interventions, all trust level changes, all governance checks.

## 5. MUTATION ZONING

- **Kernel zone**: NEVER self-modifiable. Includes this file, governance.json, permissions, kill switch.
- **Identity zone**: SOUL.md core personality + MEMORY.md North Star section + AGENTS.md agent topology. Agent may propose changes; user must manually approve and apply.
- **Skill zone**: Prompts, triggers, routing, MEMORY.md beliefs/skills/episodes/lead-sources sections. Self-modifiable with IRB score > 0.9.
- **Knowledge zone**: Books, principles, heuristics. Self-modifiable with IRB score > 0.7 and weekly integration gate.

## 6. KILL SWITCH

- File-based: `.openclaw/KILL` relative to workspace root. If exists, all agent activity halts.
- Path resolution: `%OPENCLAW_WORKSPACE%/.openclaw/KILL` (Windows) or `$OPENCLAW_WORKSPACE/.openclaw/KILL` (Unix).
- All heartbeat loops check kill switch before every action.

## 7. INTELLECTUAL GROWTH GUARDRAILS

> **Phase 3 only. Not active during Phase 0/1.**

- New principles may not be integrated during active sprints or revenue-critical periods.
- Maximum 1 principle integrated per week.
- 14-day stabilization lock after each integration.
- Book-derived insights must never modify Kernel or Identity zones.

## 8. REPAIR SHOP FILTER (Primary Veto)

Before any task gets prioritized, it must pass:

> "Does this save a technician at the bench time, or directly contribute to $250k profit target?"

If no -> deprioritize. This filter overrides intellectual novelty, cool-tech appeal, and framework enthusiasm.

## 9. LEAD SLA ENFORCEMENT

- **LeadCatcher response time**: Maximum 10 minutes from lead creation to drafted response.
- **A lead that waits 1 hour is a lost $100+ repair.**
- Lead SLA breach is always a high-priority trigger, even during focus windows.
- At L2+, the system drafts and pings. At L3, it sends (low-risk categories only).
- If multiple leads arrive simultaneously (marketing blast), queue and process in order. See `proactive/triggers.yaml` overflow settings.
- Re-escalation: If no response to initial ping within 20 min, send one re-escalation. Maximum 2 pings per lead. See `docs/FAILSAFE.md`.

## 10. TECHNICIAN FRICTION STORE

- Any bench-level friction reported by a technician (slow tool, missing part, workflow gap) is logged to `memory/friction.jsonl`.
- Friction entries with `frequency: daily` or `per_repair` are auto-elevated to L2 Draft fix priority.
- Annual cost estimate is computed: `frequency x time_lost_minutes x hourly_rate`.
- Friction fixes that save > $500/year in technician time get top priority in sprint planning.

## 11. ERROR RECOVERY

- All triggers must have defined failure paths. See `docs/FAILSAFE.md` for degradation tiers.
- Circuit breaker: 3 consecutive failures -> pause proactive actions, alert Shuki, auto-resume after 2 hours at 50% throttle.
- Partial execution: checkpoint system tracks progress. Stale runs are summarized and reported.
