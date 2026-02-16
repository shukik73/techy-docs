# Iron Secretary V2 — Security Audit Report

**Auditor:** Jay (Squad Lead)
**Date:** 2026-02-16
**Repo:** `shukik73/Iron-Secretary-V2`
**Status:** BLOCKERS ADDRESSED IN CODE — DEPLOYMENT VERIFICATION REQUIRED

---

## Executive Summary

Both CRITICAL security blockers (SS-001 and SS-002) have been **fixed in the codebase**. However, deployment status to production Supabase is unknown. This audit recommends verifying migrations have been applied before any customer onboarding.

| Issue | Status | Risk if Not Deployed |
|-------|--------|---------------------|
| SS-001: Missing RLS | ✅ Fixed in migrations | CRITICAL — Full data exposure |
| SS-002: API key exposed | ✅ Fixed in vite.config.ts | CRITICAL — API key theft |

---

## SS-001: Missing RLS on Supabase Tables

### Original Finding
The 2026-02-06 audit (AUDIT_REPORT.md) found **no RLS on any database tables** — Grade D security.

### Current Status: FIXED ✅

**30 tables identified.** All have:
- `ENABLE ROW LEVEL SECURITY` statements
- `CREATE POLICY` statements enforcing `auth.uid() = user_id`

### Tables with RLS (verified in migrations):

| Migration | Tables |
|-----------|--------|
| 20260206000001 | voice_logs |
| 20260206000002 | workspace_tasks, workspace_activity |
| 20260206000003 | night_shift_tasks, night_shift_recurring |
| 20260206000004 | Re-applies RLS + adds user_id columns + indexes |
| 20260206000005 | emilio_leads, emilio_emails, emilio_metrics, reviewguard_customers, reviewguard_reviews, reviewguard_mrr, midas_deals, midas_purchases, leads, repairs, plan_phases, plan_tasks, plan_weekly_metrics, plan_monthly_metrics, memory_events, entities, reminders |
| 20260209000001 | agent_tasks, agent_messages, agent_context, audit_logs, projects |
| 20260209000002 | notification_queue, briefing_log, telegram_config |

### RLS Policy Pattern (consistent across all tables):
```sql
CREATE POLICY "Users own data" ON table_name 
  FOR ALL 
  USING (auth.uid() = user_id);
```

### Action Required
1. **Verify migrations applied:** Run `supabase migration list` in the Supabase dashboard or CLI
2. **Check production:** Query `pg_policies` table to confirm policies exist
3. **Test:** Attempt cross-user data access with anon key (should fail)

---

## SS-002: API Key Exposed in Client Bundle

### Original Finding
The 2026-02-06 audit found Gemini API key injected into client bundle via vite.config.ts:
```js
// DANGEROUS — REMOVED
'process.env.API_KEY': JSON.stringify(env.GEMINI_API_KEY),
'process.env.GEMINI_API_KEY': JSON.stringify(env.GEMINI_API_KEY)
```

### Current Status: FIXED ✅

**vite.config.ts now contains:**
```ts
// API keys must NEVER be injected into the client bundle.
// Use Supabase Edge Functions or a backend proxy for AI API calls.
// Client-side env vars use VITE_ prefix and are accessed via import.meta.env.
```

No API key injection code remains. AI API calls should route through:
- `supabase/functions/ai-chat/index.ts` (uses `Deno.env.get('OPENAI_API_KEY')`)
- `supabase/functions/parse-utterance/index.ts` (uses `Deno.env.get('OPENAI_API_KEY')`)
- `supabase/functions/ocr-extract/index.ts` (uses `Deno.env.get('OPENAI_API_KEY')`)

### Action Required
1. **Verify Edge Functions deployed:** Check Supabase dashboard → Edge Functions
2. **Confirm secrets set:** `OPENAI_API_KEY` must be set in Supabase project secrets
3. **Rotate keys:** If the old Gemini key was ever in a deployed build, rotate it

---

## Additional Vulnerabilities Found

### Medium Priority

1. **Tailwind CDN in production** (`index.html:8`)
   - Using `cdn.tailwindcss.com` is explicitly warned against by Tailwind team
   - Risk: Supply chain attack vector
   - Fix: Bundle Tailwind locally (already have `@tailwindcss/vite` plugin configured)

2. **No Content Security Policy**
   - No CSP headers defined
   - Risk: XSS attacks
   - Fix: Add CSP meta tag or configure headers in Vercel/hosting

3. **esm.sh runtime dependency loading** (`index.html:94-103`)
   - All deps loaded from esm.sh at runtime
   - Risk: CDN compromise breaks entire app
   - Fix: Bundle dependencies locally

### Low Priority

4. **No input validation on forms**
   - Task titles, descriptions, prompts have no length limits
   - Risk: DoS via mega-input, potential XSS when rendered
   - Fix: Add maxLength attributes, sanitize before database write

5. **Speech API error handling missing**
   - `SpeechRecognition.start()` can throw if permissions denied
   - Risk: Unhandled promise rejection
   - Fix: Wrap in try/catch

---

## Deployment Verification Checklist

Before declaring CRITICAL blockers resolved:

- [ ] Run `supabase db push` or verify migrations in Supabase dashboard
- [ ] Query production: `SELECT * FROM pg_policies;` — confirm all 30+ policies exist
- [ ] Deploy Edge Functions: `supabase functions deploy`
- [ ] Set secrets: `supabase secrets set OPENAI_API_KEY=sk-...`
- [ ] Test RLS: Try querying another user's data with anon key (should return 0 rows)
- [ ] Rotate any potentially leaked API keys

---

## Estimated Effort

| Task | Time | Who |
|------|------|-----|
| Verify migration status | 15 min | Kai/Shuki (Supabase access) |
| Apply migrations if needed | 5 min | Kai/Shuki |
| Deploy Edge Functions | 10 min | Kai/Shuki |
| Rotate API keys | 10 min | Shuki |
| Add CSP headers | 30 min | Jay (T2 when approved) |
| Bundle Tailwind locally | 20 min | Jay (T2 when approved) |

**Total to clear CRITICAL blockers:** ~40 minutes (requires Supabase access)

---

## Recommendation

**Escalate to Kai:** I cannot verify production deployment status — I only have repo access, not Supabase dashboard access.

The code fixes are complete. The blockers will be resolved once someone with Supabase access confirms:
1. Migrations have been applied
2. Edge Functions are deployed
3. Secrets are configured

Once verified, Iron Secretary is clear for customer onboarding from a security standpoint.

---

*Report generated by Jay | Squad Lead | T1 (Execute)*
