# MULTI-AGENT LEADERSHIP PROTOCOL v1.0

> **STATUS: Phase 2 — Not active. Do not load into boot context during Phase 0/1.**

## HIERARCHY

```
CEO Agent (Main Thread)
+-- Supervisor: Frontend-Lead
+-- Supervisor: Ops-Lead
+-- Workers (ephemeral)
    +-- Coder
    +-- Researcher
    +-- Reviewer
    +-- Security-Auditor
```

## TOOL BOUNDARIES (enforced, not advisory)

| Role | Allowed Tools | Forbidden Tools |
|------|--------------|-----------------|
| CEO | Plan, Delegate, Read, TodoWrite | Edit, Write, Bash (destructive) |
| Worker | Edit, Write, Bash | Delegate, Spawn |
| Reviewer | Read, Analyze | Edit, Write |

**Note**: These boundaries only apply when multi-agent mode is active (Phase 2+). During Phase 0, the single agent operates under `phase_overrides` defined in `schemas/governance.json`.

## DELEGATION RULES

1. **Stateless Spawning**: Workers are ephemeral. No shared memory beyond assigned sub-directory.
2. **Context Paging**: Inject only the architecture plan + current file context. Never flood workers with the full repo.
3. **Locking Protocol**: Workers must acquire lockfile in `.openclaw/locks/` before editing shared source files. Release on completion or timeout (5 min).
4. **Single Lane**: One worker per file at a time. Queue if contended.

## INTERNAL REVIEW BOARD (Phase 2+)

For critical actions (deployments, credential updates, external communications):

1. **Resolver Agent**: Proposes the solution
2. **Detector Agent**: Identifies gaps, edge cases, security issues
3. **Evaluator Agent**: Final quality score (threshold > 0.9 for auto-execution)

If score < 0.9: escalate to user with analysis from all three agents.

**Cost note**: Running 3 agents triples token cost. Only invoke full IRB for risk >= high. For risk == medium, use Resolver + single Reviewer pass.

## ADVERSARIAL REVIEW

- Reviewer agent is prompted to be critical and security-focused
- Reviewer MUST flag: hardcoded secrets, missing error handling, SQL injection vectors, unvalidated inputs, missing auth checks
- Coder's output is NEVER accepted without Reviewer pass in risk >= medium tasks
