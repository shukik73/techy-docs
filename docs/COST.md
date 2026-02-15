# COST AWARENESS LAYER v1.0

## BUDGETS

| Scope | Max USD |
|-------|---------|
| Per trigger | $0.20 |
| Per hour | $0.50 |
| Per day | $5.00 |
| Per month | $100.00 |

## MODEL ROUTING

| Task Type | Model Tier | Examples |
|-----------|-----------|---------|
| Classification, triage | Cheap (local 8B / Haiku) | "Is this a lead or spam?", log categorization |
| Summarization, drafts | Medium (Sonnet) | Morning brief, review response drafts |
| Code review, architecture | Strong (Sonnet/Opus) | PR review, security audit |
| Executive decisions | Strong (on-demand only) | Strategic sparring, drift analysis |

## DECISION GATE

Execute proactive step ONLY IF:

```
ExpectedFutureCost > InterventionCost x (1 + AnnoyanceRisk)
```

Where:
- AnnoyanceRisk = 0.5 during focus hours
- AnnoyanceRisk = 0.1 otherwise
- AnnoyanceRisk = 0.8 if user said "stop" in last 2 hours

## EXPECTED FUTURE COST — Phase 0 Lookup Table

For Phase 0, use these pre-computed values instead of dynamic calculation:

| Trigger | ExpectedFutureCost (USD) | Rationale |
|---------|-------------------------|-----------|
| lead_sla_breach | $150.00 | Average repair ticket value |
| ci_failure_triage | $25.00 | Developer time to manually debug |
| morning_brief | $10.00 | Decision latency cost of uninformed day |
| workspace_health | $5.00 | Infrastructure downtime prevention |
| review_response_needed | $50.00 | Customer lifetime value at risk |
| strategic_drift_flag | $20.00 | Opportunity cost of misaligned week |
| re_escalation | $200.00 | Lead has been waiting 20+ min, higher loss probability |

These values should be calibrated after 30 days of operation based on actual outcomes.

## TRACKING

Every episode logs:
- `tokens_in`, `tokens_out`, `usd_est`
- Accumulated in `memory/cost_ledger.jsonl`

Heartbeat checks hourly budget before executing any trigger. If budget exhausted: log observation only, no LLM calls.

## BUDGET ENFORCEMENT

Budget enforcement is handled by the orchestration layer (heartbeat.mjs / n8n), NOT by the LLM itself. The orchestration layer:
1. Reads `memory/cost_ledger.jsonl` before each trigger execution
2. Sums costs for the current hour/day/month
3. Blocks LLM calls if any budget ceiling is hit
4. Logs a `budget_exhausted` event and notifies Shuki via Telegram if daily budget is reached
