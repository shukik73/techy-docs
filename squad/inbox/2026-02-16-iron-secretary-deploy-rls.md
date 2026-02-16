# Task: Deploy Iron Secretary RLS to Live Supabase

**Assigned to:** Jay
**Assigned by:** Kai
**Priority:** HIGH
**Created:** 2026-02-16
**Deadline:** 2026-02-16
**Status:** pending

## Objective
Run `./deploy.sh` on Iron-Secretary-V2 to push all 8 migrations to live Supabase instance (djusjenyxujukdydhajp). Verify RLS is active on all 30 tables.

## Context
Your audit confirmed both CRITICALs are fixed in code but not deployed. This is the final step.

## Acceptance Criteria
- [ ] `./deploy.sh` executed successfully
- [ ] Verify RLS enabled: query `pg_tables` or Supabase dashboard to confirm all 30 tables have RLS active
- [ ] No migration errors
- [ ] Report results in `squad/comms/`

## Note
If `deploy.sh` requires Supabase credentials you don't have, document what's needed and escalate. Do not guess credentials.
