# OpenClaw Executive Architecture — Kai v1.0

Proactive, memory-aware autonomous operator for Techy Miramar.

## Directory Layout

```
openclaw-executive/
├── SYSTEM_PROMPT.md              # Boot sequence — injected as system prompt
├── kernel/
│   └── POLICY.md                 # IMMUTABLE governance guardrails (Layer 0)
├── docs/
│   ├── SOUL.md                   # Executive personality + stability doctrine (Layer 1)
│   ├── MEMORY.md                 # Belief summary + North Star (Layer 2)
│   ├── COST.md                   # Token budgets and routing rules
│   ├── FAILSAFE.md               # Error recovery and degradation tiers
│   ├── RUNTIME.md                # How Kai actually executes (execution model)
│   ├── ONBOARDING_MESSAGE.md     # First-boot onboarding message from Shuki to Kai
│   ├── INTELLECT.md              # Intellectual growth protocol (Phase 3 — not active)
│   ├── AGENTS.md                 # Multi-agent protocol (Phase 2 — not active)
│   └── EVOLVE.md                 # Self-improvement + assertiveness (Phase 3 — not active)
├── schemas/
│   ├── governance.json           # Kernel config with mutation zones (Layer 0)
│   ├── belief.schema.json        # Belief object spec
│   ├── episode.schema.json       # Episode log spec
│   ├── intervention.schema.json  # Proactive intervention spec
│   ├── checkpoint.schema.json    # Crash recovery spec
│   ├── book.schema.json          # Book distillation spec
│   ├── assertiveness.schema.json # Assertiveness calibration spec
│   └── friction.schema.json      # Technician friction entry spec
├── memory/
│   ├── beliefs.jsonl             # Append-only belief events (audit trail)
│   ├── beliefs.snapshot.json     # Materialized belief state (fast reads)
│   ├── episodes.jsonl            # Runtime episode log
│   ├── cost_ledger.jsonl         # Token spend tracking
│   └── friction.jsonl            # Technician friction reports
├── knowledge/
│   ├── books/                    # Distilled book extractions
│   ├── principles.json           # Active integrated principles
│   ├── heuristics.json           # Decision heuristics
│   └── reading_queue.json        # Prioritized reading list (Phase 3)
├── metrics/
│   └── assertiveness.json        # Current assertiveness state
├── proactive/
│   ├── triggers.yaml             # Trigger definitions + trust ramp + budgets
│   └── interventions.jsonl       # Intervention audit log
├── workflows/
│   ├── lead-sla-guardian.json    # n8n workflow: Lead intake + Claude draft + Telegram ping
│   └── lead-sla-escalation.json  # n8n workflow: 10-min SLA breach escalation
└── runs/
    ├── <run_id>/state.json       # Per-run checkpoints for crash recovery
    ├── undelivered/              # Failed notifications queued for retry
    └── config_backups/           # Last-known-good config snapshots
```

## Architecture Layers

| Layer | Name | Files | Phase |
|-------|------|-------|-------|
| 0 | Immutable Kernel | governance.json, POLICY.md | Active |
| 1 | Executive Identity | SOUL.md | Active |
| 2 | Belief Engine | beliefs.snapshot.json, belief.schema.json | Active |
| 3 | Anticipation Engine | triggers.yaml | Active |
| 4 | Multi-Agent Orchestration | AGENTS.md | Phase 2 |
| 4.5 | Intellectual Growth | INTELLECT.md, book.schema.json | Phase 3 |
| 5 | Self-Improvement | EVOLVE.md, assertiveness.schema.json | Phase 3 |

## Phase 0 Boot Context (What Gets Loaded)

Only these files are injected into the LLM context during Phase 0:

1. `kernel/POLICY.md` — Governance guardrails
2. `docs/SOUL.md` — Personality and assertiveness bounds
3. `docs/MEMORY.md` — North Star + beliefs + vendor rules
4. `docs/COST.md` — Token budgets and model routing
5. `docs/RUNTIME.md` — Execution model (stateless LLM, n8n, tool access)
6. `proactive/triggers.yaml` — Trigger definitions
7. `memory/beliefs.snapshot.json` — Current belief state
8. `metrics/assertiveness.json` — Current assertiveness level
9. `docs/FAILSAFE.md` — Error recovery tiers

**NOT loaded in Phase 0** (saves ~3000 tokens per invocation):
- `docs/INTELLECT.md` — Phase 3
- `docs/EVOLVE.md` — Phase 3
- `docs/AGENTS.md` — Phase 2
- `knowledge/reading_queue.json` — Phase 3

## First Live Workflow: Lead SLA Guardian

```
Lead arrives (RepairDesk/HubSpot/website)
    -> n8n webhook receives POST
    -> Normalize lead data
    -> Claude drafts SMS response
    -> Telegram ping to Shuki with draft
    -> 10-min timer starts
    -> If no response: escalation alert
    -> If still no response at 20 min: re-escalation (max 2 pings)
```

See `docs/RUNTIME.md` for how Kai executes, and `SETUP.md` for configuration.

## Phased Roadmap

### Phase 0 — Executive MVP (current)
- [x] Belief store schema + update rules
- [x] Episode log schema
- [x] Trust ramp (starts at L1 = suggest only)
- [x] Cost awareness layer with Phase 0 lookup table
- [x] Kill switch + governance kernel
- [x] Checkpoint/recovery schema
- [x] Phase 0 tool override (single-agent permissions)
- [x] Error recovery / FAILSAFE.md
- [x] Runtime execution model / RUNTIME.md
- [ ] Wire to real trigger (Lead SLA Guardian via n8n)
- [ ] heartbeat.mjs implementation

### Phase 1 — Operational Orchestration (week 1)
- [ ] Connect heartbeat to real signal sources (RepairDesk webhook, GitHub Actions)
- [ ] Implement LLM call routing (cheap vs strong model)
- [ ] Single reviewer sub-agent for code/draft review
- [ ] Gateway crash recovery (reload checkpoint, resume or summarize)

### Phase 2 — Swarm Executive (weeks 2-3)
- [ ] CEO/Worker tool-boundary enforcement in gateway
- [ ] Lane queue + lockfiles for parallel workers
- [ ] Adversarial review board (Resolver/Detector/Evaluator)
- [ ] Multi-step project completion with <1 user clarification

### Phase 3 — Recursive Improvement (month 2)
- [ ] Activate INTELLECT.md and EVOLVE.md in boot sequence
- [ ] Mutation sandbox with Protected Kernel Doctrine
- [ ] Benchmark suite for skills
- [ ] Propose -> Simulate -> Score -> Adopt pipeline
- [ ] Weekly drift detection with auto-pause

## Kill Switch

Create `.openclaw/KILL` in the workspace root to immediately halt all agent activity:

```bash
# Stop (Unix)
touch .openclaw/KILL
# Stop (Windows)
echo. > .openclaw\KILL

# Resume (Unix)
rm .openclaw/KILL
# Resume (Windows)
del .openclaw\KILL
```

## Key Design Decisions

1. **Profit, not revenue**: North Star is $250K annual net profit, tracked as a rolling annual target
2. **Repair Shop Filter**: Primary veto — every task must justify bench-time savings or profit contribution
3. **Trust Ramp**: Starts at L1 (suggest only), promotes through proven interventions
4. **Phase 0 tool override**: Single agent gets expanded permissions with full audit logging until Phase 2 workers are available
5. **Budget enforcement in code**: Token budgets are enforced by heartbeat.mjs / n8n, not by LLM self-regulation
6. **Graceful degradation**: Every trigger has defined failure paths (see FAILSAFE.md)
