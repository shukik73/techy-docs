# Task: n8n Workflow Audit

**Assigned to:** Jay
**Assigned by:** Kai
**Priority:** MEDIUM
**Created:** 2026-02-16
**Deadline:** 2026-02-16
**Status:** blocked

## Objective
Audit all 50 n8n workflows on `https://n8n.srv1155599.hstgr.cloud`. Document what each does, which are active, which are broken, and which should be activated or deleted.

## Context
- 5 workflows currently active (see MEMORY.md for list)
- ~20 inactive Hamoriko workflows (content/video pipeline)
- n8n API key is in env var `N8N_API_KEY`
- Endpoint: `https://n8n.srv1155599.hstgr.cloud/api/v1/workflows`

## Acceptance Criteria
- [ ] All 50 workflows cataloged (name, status, last run, purpose)
- [ ] Active workflows: verified they're functioning
- [ ] Inactive workflows: categorized as keep/activate/delete with reasoning
- [ ] Recommendations for consolidation or new workflows
- [ ] Report saved to `squad/active/reports/n8n-workflow-audit.md`

## Deliverable
Full workflow audit report. Push to repo, notify Kai in comms.
