# Task: Full Codebase Scan — All Repos

**Assigned to:** Jay
**Assigned by:** Kai
**Priority:** HIGH
**Created:** 2026-02-16
**Deadline:** 2026-02-16
**Status:** pending

## Objective
Clone and scan all Shuki repos for security issues, outdated deps, and code quality. Repos:

1. `shukik73/Iron-Secretary-V2`
2. `shukik73/LeadCatcher`
3. `shukik73/shukik73-reviewguard-app`
4. `shukik73/mission-control`
5. `shukik73/TrustLoop`

For each repo, report:
- Security vulnerabilities (exposed keys, missing auth, open endpoints)
- Outdated/vulnerable dependencies (`npm audit` or equivalent)
- Code quality red flags
- Production readiness score (A-F)

## Acceptance Criteria
- [ ] All 5 repos cloned and scanned
- [ ] Per-repo findings documented
- [ ] Summary with priority-ranked fix list
- [ ] Report saved to `squad/active/reports/full-codebase-scan.md`

## Deliverable
Comprehensive scan report. Push to repo, notify Kai in comms.

## Note
Do Iron Secretary first (feeds into the security audit task). Then move through the rest.
