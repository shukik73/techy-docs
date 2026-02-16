# COST.md — Agent Squad Budget & Spend Tracking
*Last updated: 2026-02-15 | Owner: Kai | Enforced by: Jay*

## Monthly Budget

| Category | Monthly Cap | Notes |
|----------|-------------|-------|
| **Total Agent Ops** | $150/mo | All AI API calls, spawned agents, token usage |
| **Per-task cap** | $2.00 | Single task max before escalation to Kai |
| **Scout agent** | $50/mo | Parts searching, price monitoring, API calls |
| **Worker agents (other)** | $50/mo | Pixel, Buzz, Midas, Pluto, Emilio, Secretary |
| **Jay overhead** | $30/mo | Jay's own operations, coordination, Mission Control checks |
| **Reserve** | $20/mo | Unplanned, requires Kai approval per use |

## Per-Call Estimates (Reference)

| Operation | Est. Cost | Notes |
|-----------|-----------|-------|
| Claude Sonnet turn (Jay) | ~$0.01–0.03 | Standard task execution |
| Claude Opus turn (Kai) | ~$0.05–0.15 | Strategic decisions |
| Spawn sub-agent (short task) | ~$0.05–0.20 | Depends on model + turns |
| Spawn sub-agent (complex) | ~$0.50–2.00 | Multi-step research/analysis |
| Web search (Brave) | ~$0.005 | Per query |
| Web fetch | ~$0.00 | Free (no API cost) |
| eBay API call | TBD | Not yet connected |
| Amazon API call | TBD | Not yet connected |

## Escalation Thresholds

| Threshold | Action |
|-----------|--------|
| Task > $2.00 | **Stop.** Escalate to Kai before continuing. |
| Daily spend > $10 | Alert Kai in group chat. |
| Monthly 80% burn ($120) | Kai reviews remaining budget, may restrict. |
| Monthly 100% burn ($150) | **Hard stop.** No agent ops without Shuki approval. |
| Any new recurring cost | Kai approval required before committing. |

## Spend Tracking

All spend logged to `memory/cost_ledger.jsonl` with format:
```json
{"ts":"ISO-8601","agent":"jay|scout|pixel|...","task":"description","model":"model-used","tokens_in":0,"tokens_out":0,"est_cost":0.00,"category":"ops|search|worker|reserve"}
```

### February 2026

| Date | Agent | Task | Est. Cost | Running Total |
|------|-------|------|-----------|---------------|
| — | — | *No spend logged yet* | — | $0.00 |

**MTD Total: $0.00 / $150.00**

## Quarterly Review

| Quarter | Budget | Actual | Status |
|---------|--------|--------|--------|
| Q1 2026 (Jan–Mar) | $450 | TBD | In progress |
| Q2 2026 (Apr–Jun) | $450 | — | Planned |

## Rules for Jay

1. **Log every spend** — if it's not in the ledger, it didn't happen.
2. **Batch over blast** — one well-crafted call beats five lazy ones.
3. **Cheapest model that works** — don't use Opus for a Sonnet job.
4. **No open-ended loops** — every agent spawn must have a timeout and max turns.
5. **Scout gets the biggest slice** because parts sourcing directly drives revenue. Other workers share the rest.
6. **If budget is tight, prioritize:** Scout > Secretary > others. Revenue-generating agents eat first.

## Mission Control Display

This data feeds into the Mission Control dashboard cost panel:
- **Gauge:** MTD spend vs. monthly cap (green/yellow/red at 60%/80%/100%)
- **Table:** Last 10 transactions from cost_ledger.jsonl
- **Trend:** Daily spend over rolling 30 days
- **Alerts:** Any threshold breach surfaces as a banner

Dashboard endpoint: `GET /api/costs` (to be wired)
