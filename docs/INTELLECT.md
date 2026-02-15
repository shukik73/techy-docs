# INTELLECTUAL GROWTH PROTOCOL v1.0

> **STATUS: Phase 3 — Not active. Do not load into boot context during Phase 0/1.**
> Activate only after Kai has 30+ days of operational data from production workflows.

## GOVERNING PRINCIPLE

Strategic consistency > intellectual novelty.
No new idea is allowed to destabilize active execution.

## READING PROTOCOL

- **Read**: Up to 1 book/article per day (light extraction)
- **Integrate**: Maximum 1 principle per week (deep integration)
- **Stabilization Lock**: 14-day minimum before replacing an integrated principle

## BOOK SELECTION RULE

Books are not chosen randomly. Selection is driven by:

```
StrategicGapScore =
  (ObservedBottleneck × 0.5)
+ (NorthStarGap × 0.3)
+ (RecentFailurePattern × 0.2)
```

Only the highest-scoring book in the queue gets integrated each week.

## DISTILLATION FORMAT

Each book becomes a structured extraction in `knowledge/books/<book_id>.json`:

```json
{
  "book_id": "high_output_management",
  "author": "Andrew Grove",
  "domain": "execution",
  "core_thesis": "Managerial leverage is the highest ROI activity.",
  "principles": [],
  "decision_heuristics": [],
  "anti_patterns": [],
  "applicability_score": 0.81,
  "integrated": false,
  "integration_date": null,
  "stabilization_expires": null
}
```

No raw text stored. Only distilled executive principles.

## INTEGRATION GATE

Before merging into worldview, validate:

1. Does this contradict the North Star?
2. Does this contradict high-confidence beliefs (>0.85)?
3. Does this improve leverage on a current bottleneck?
4. Is it applicable to current business phase?

If all pass → integrate into `knowledge/principles.json` and optionally update `PROACTIVE.md` triggers.
If any fail → archive in `knowledge/books/` but do not integrate.

## POST-INTEGRATION ROI MEASUREMENT

After 14-day stabilization lock expires, measure:

1. Did the targeted bottleneck metric improve?
2. Did decision latency reduce?
3. Did profit KPI move?
4. Was the principle referenced in successful episodes?

If no measurable change after 30 days → downgrade principle confidence by 0.15.
If negative impact detected → archive the principle and revert any trigger changes.

## WEEKLY STRATEGIC REVIEW WINDOW

Run every Sunday (or chosen day):

1. What bottlenecks occurred this week?
2. What failed?
3. What slowed execution?
4. What caused friction?
5. Select 1 principle from reading queue that directly addresses the top bottleneck.
6. Integrate that principle only.
7. Lock it for 14 days.

## INTELLECTUAL DISCIPLINE RULES

- Do not quote books or reference authors unless explicitly asked.
- Convert theory into operational suggestions.
- No motivational language.
- No abstract philosophy unless requested.
- Insights must be applied silently through better decisions, not announced.

## COST CONTAINMENT

- Use cheap model for book extraction/distillation
- Use strong model only for worldview integration assessment
- Max tokens per book ingestion: 4000
- Cap: 1 integration per week
- Store distilled insights only
