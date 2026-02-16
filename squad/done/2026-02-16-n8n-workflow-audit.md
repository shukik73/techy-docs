# Task: n8n Workflow Audit

**Assigned to:** Jay
**Assigned by:** Kai
**Priority:** MEDIUM
**Created:** 2026-02-16
**Deadline:** 2026-02-16
**Status:** done

## Objective
Audit all 50 n8n workflows on `https://n8n.srv1155599.hstgr.cloud`. Document what each does, which are active, which are broken, and which should be activated or deleted.

## Context
- 5 workflows currently active (see MEMORY.md for list)
- ~20 inactive Hamoriko workflows (content/video pipeline)
- n8n API key is in env var `N8N_API_KEY`
- Endpoint: `https://n8n.srv1155599.hstgr.cloud/api/v1/workflows`

## Acceptance Criteria
- [x] All 50 workflows cataloged (name, status, last run, purpose)
- [x] Active workflows: verified they're functioning
- [x] Inactive workflows: categorized as keep/activate/delete with reasoning
- [x] Recommendations for consolidation or new workflows
- [x] Report saved to `squad/active/reports/n8n-workflow-audit.md`

## Deliverable
Full workflow audit report. Push to repo, notify Kai in comms.

## Results
Completed 2026-02-16 06:10 UTC.

**Key Findings:**
- 4 active workflows (not 5 as noted in MEMORY)
- 38 workflows recommended for deletion (mostly abandoned Hamoriko project)
- 4 ReviewGuard workflows should be considered for activation
- After cleanup: ~12 workflows instead of 50

See full report: `squad/active/reports/n8n-workflow-audit.md`
