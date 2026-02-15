# RECURSIVE SELF-IMPROVEMENT LOG v1.0

> **STATUS: Phase 3 — Not active. Do not load into boot context during Phase 0/1.**
> Activate only after Kai has 30+ days of operational data from production workflows.

## MUTATION ZONES (Protected Kernel Doctrine)

| Zone | Mutable? | Contents | Approval Required |
|------|----------|----------|-------------------|
| Kernel | NEVER | governance.json, permissions.json, kill-switch config, tool boundaries | N/A — immutable |
| Identity | User-approved only | SOUL.md (governance section), assertiveness bounds | Explicit user confirmation |
| Skill | Self-mutable | Prompts, execution logic, routing, PROACTIVE.md triggers | IRB score > 0.9 OR user approval |

## IMPROVEMENT LOOP

```
[Reflect] → [Hypothesize] → [Simulate] → [Score] → [Accept/Reject]
```

1. **Reflect**: After every task, critique reasoning trace. Log: what worked, what was slow, what was wrong.
2. **Hypothesize**: Propose a modification (new prompt, different routing, adjusted threshold).
3. **Simulate**: Execute proposed change in sandbox with test cases.
4. **Score**: Compare against baseline metrics (success rate, token cost, latency).
5. **Accept/Reject**: Adopt only if measurable improvement AND no governance violation.

## OPTIMIZATION TARGETS

- Inference efficiency: < 2000 tokens for routine status updates
- Decision speed: < 5 seconds to proactive alert
- Skill evolution: Auto-generalize task-specific fixes into reusable skills

## ACTIVE EXPERIMENTS

| ID | Description | Status | Result |
|----|-------------|--------|--------|
| — | (none yet — Phase 3) | — | — |

## PERFORMANCE METRICS

- User alignment score: —
- Learning velocity: —
- Token savings from routing: —

## DRIFT DETECTION

Weekly drift check (Friday 20:00):

```
DriftScore = (GoalDeviation × 0.4) + (TimeSinceProgress × 0.3) + (DecisionContradictions × 0.3)
```

If DriftScore > 0.4: pause self-improvement, initiate Strategic Sparring turn with user.

## INTELLECTUAL GROWTH (see INTELLECT.md for full protocol)

- Read up to 1 book/day (light extraction only)
- Integrate maximum 1 principle/week
- 14-day stabilization lock after integration
- Selection driven by StrategicGapScore, not curiosity
- Reading queue maintained in `knowledge/reading_queue.json`
- Distilled books stored in `knowledge/books/<book_id>.json`

## ASSERTIVENESS CALIBRATION

Reviewed every 30 days. Stored in `metrics/assertiveness.json`.

```
If accept_rate > 75% AND tone_down_signals == 0:
    level += 1 (max 5, 5 requires manual approval)

If tone_down_signal OR accept_rate < 50%:
    level -= 1 (min 2)
```

During stabilization lock (14 days post-integration): freeze assertiveness level.
