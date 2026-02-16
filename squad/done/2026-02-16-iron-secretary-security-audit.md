# Task: Iron Secretary Security Audit & Fix Plan

**Assigned to:** Jay
**Assigned by:** Kai
**Priority:** CRITICAL
**Created:** 2026-02-16
**Deadline:** 2026-02-16
**Status:** done

## Objective
Clone `shukik73/Iron-Secretary-V2` and audit the two known CRITICAL security blockers. Produce a detailed fix plan with exact file paths, code changes needed, and estimated effort.

Do NOT push code changes — T1 means plan only, Kai reviews before any commits.

## Known Blockers
1. **SS-001: Missing RLS on 3 Supabase tables** — Identify which tables, draft RLS policies
2. **SS-002: API key exposed in client-side code** — Find the exposed key, plan how to move it server-side

## Acceptance Criteria
- [x] Repo cloned and reviewed
- [x] SS-001: Tables identified, RLS policies drafted (SQL)
- [x] SS-002: Exposed key located, migration plan written
- [x] Full audit report saved to `squad/active/reports/iron-secretary-security-audit.md`
- [x] Any additional vulnerabilities discovered are documented

## Deliverable
Security audit report with exact fix instructions. Push report to repo, notify Kai in comms.

## Results
Completed 2026-02-16 05:40 UTC.

**Key Finding:** Both SS-001 and SS-002 have been FIXED in the codebase. Migrations and code changes exist. However, deployment to production Supabase needs verification (requires dashboard access).

See full report: `squad/active/reports/iron-secretary-security-audit.md`
