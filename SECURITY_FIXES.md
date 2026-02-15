# Security Fixes Roadmap — All 3 Codebases

**Compiled:** 2026-02-11  
**Total Blockers:** 5  
**Total Time to Fix:** ~7-9 hours  
**Estimated ROI:** Prevents data breach, credential compromise, injection attacks  

---

## Master Priority List (Ordered by Risk → Impact)

### CRITICAL BLOCKERS (Must Fix Before Production)

**Total: 5 blockers | Fix time: ~7-9h | ROI: Prevent data breach + credential theft**

| ID | Codebase | Issue | Severity | Time | Status |
|---|---|---|---|---|---|
| SS-001 | Iron Secretary V2 | Supabase RLS missing on 3 tables | CRITICAL | 2h | Blocked |
| SS-002 | Iron Secretary V2 | Gemini API key exposed in client bundle | CRITICAL | 15m | Blocked |
| SS-003 | LeadCatcher | Twilio webhook signature validation missing | HIGH | 1h | Blocked |
| SS-004 | LeadCatcher | Open redirect in auth callback (`next` param) | HIGH | 1h | Blocked |
| SS-005 | LeadCatcher | Settings page crash (fetchSettings before declaration) | HIGH | 15m | Blocked |

---

## Detailed Fix Guide

### Iron Secretary V2

#### SS-001: Missing RLS Policies on Supabase Tables
**File:** Supabase dashboard (not in codebase)  
**Severity:** CRITICAL  
**Time:** 2h  
**Risk:** Any authenticated user can read all voice_logs, workspace_tasks, night_shift_tasks

**Affected Tables:**
1. `voice_logs` — All calls + transcriptions (PII)
2. `workspace_tasks` — All AI workspace data
3. `night_shift_tasks` — All night shift orchestration data

**Fix Steps:**

```sql
-- 1. Enable RLS on all 3 tables
ALTER TABLE voice_logs ENABLE ROW LEVEL SECURITY;
ALTER TABLE workspace_tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE night_shift_tasks ENABLE ROW LEVEL SECURITY;

-- 2. Create policy: Users can only read/update their own data
CREATE POLICY "Users can read own voice logs"
  ON voice_logs
  FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own voice logs"
  ON voice_logs
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own voice logs"
  ON voice_logs
  FOR UPDATE
  USING (auth.uid() = user_id);

-- 3. Repeat for workspace_tasks and night_shift_tasks
-- Copy the pattern above, replace table name and user_id column
```

**Verification:**
```bash
# Log into Supabase as non-owner user
# Try to query voice_logs → should return empty
SELECT * FROM voice_logs WHERE user_id != auth.uid();
-- Result: 0 rows (policy enforced ✓)
```

**Deliverable:** Screenshot of RLS policies enabled in Supabase dashboard

---

#### SS-002: Gemini API Key Exposed in Client Bundle
**File:** `vite.config.ts:12-13`  
**Severity:** CRITICAL  
**Time:** 15m  
**Risk:** API key extractable from browser DevTools → anyone can call Gemini API as you

**Current Code:**
```typescript
// vite.config.ts
export default defineConfig({
  define: {
    'process.env.VITE_GEMINI_API_KEY': JSON.stringify(process.env.GEMINI_API_KEY)
  }
})
```

**Problem:**
- GEMINI_API_KEY injected into client JavaScript
- Browser can extract key from `window.__VITE_ENV__` or bundle
- Attacker can make unlimited Gemini API calls on your dime

**Fix:**

**OPTION A: Backend Proxy (Recommended)**
```typescript
// vite.config.ts — REMOVE this:
// 'process.env.VITE_GEMINI_API_KEY': JSON.stringify(...)

// Instead, create backend endpoint:
// backend/routes/aiRoutes.js
router.post('/api/ai/chat', requireAuth, async (req, res) => {
  const { message } = req.body;
  
  // Backend holds Gemini key
  const gemini = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
  const model = gemini.getGenerativeModel({ model: 'gemini-pro' });
  
  const result = await model.generateContent(message);
  res.json({ response: result.response.text() });
});

// Frontend calls /api/ai/chat instead of Gemini directly
```

**OPTION B: Environment Variable Prefix (If must stay frontend)**
```typescript
// Only expose non-sensitive keys
define: {
  'process.env.VITE_PUBLIC_API_URL': JSON.stringify(process.env.VITE_PUBLIC_API_URL),
  'process.env.VITE_PUBLIC_STRIPE_KEY': JSON.stringify(process.env.VITE_PUBLIC_STRIPE_KEY)
  // Do NOT expose VITE_GEMINI_API_KEY
}
```

**Verification:**
```bash
# Build and check bundle
npm run build
# Search bundle for API key
grep -r "AIza" dist/
# Should return: NOTHING (✓)
```

**Deliverable:** Verify Gemini API key is NOT in `dist/` folder after build

---

### LeadCatcher

#### SS-003: Missing Twilio Signature Validation on Webhook
**File:** `/src/app/api/webhooks/twilio/transcription/route.ts`  
**Severity:** HIGH  
**Time:** 1h  
**Risk:** Attackers can inject fake transcriptions → auto-reply to spam customers

**Current Code (Vulnerable):**
```typescript
export async function POST(req: Request) {
  const body = await req.json();
  
  // No signature validation!
  const { transcription, call_sid } = body;
  
  // Directly processes transcription
  // ...
}
```

**Attack Scenario:**
```bash
# Attacker spoofs POST to your webhook
curl -X POST https://leadcatcher.app/api/webhooks/twilio/transcription \
  -H "Content-Type: application/json" \
  -d '{"transcription":"SPAM MESSAGE","call_sid":"fake"}'

# Your app processes it, sends auto-reply to innocent customer
```

**Fix:**

```typescript
import crypto from 'crypto';

// Validate Twilio signature
function validateTwilioRequest(
  token: string,
  twilioSignature: string,
  url: string,
  body: Record<string, any>
): boolean {
  // Reconstruct the request body
  const params = new URLSearchParams(body).toString();
  const data = url + params;
  
  const hash = crypto
    .createHmac('sha1', token)
    .update(data)
    .digest('base64');
  
  return hash === twilioSignature;
}

export async function POST(req: Request) {
  const twilioAuthToken = process.env.TWILIO_AUTH_TOKEN;
  const signature = req.headers.get('x-twilio-signature');
  const url = new URL(req.url).origin + new URL(req.url).pathname;
  
  const body = await req.json();
  
  // ✓ Validate signature before processing
  if (!validateTwilioRequest(twilioAuthToken, signature || '', url, body)) {
    return new Response(JSON.stringify({ error: 'Invalid signature' }), {
      status: 403
    });
  }
  
  // Safe to process transcription now
  const { transcription, call_sid } = body;
  // ...
}
```

**Verification:**
```typescript
// Unit test
const validSignature = crypto
  .createHmac('sha1', 'test-token')
  .update(url + params)
  .digest('base64');

// Should reject invalid signature
expect(validateTwilioRequest('test-token', 'invalid', url, body)).toBe(false);
```

**Deliverable:** Unit test proving signature validation works

---

#### SS-004: Open Redirect in Auth Callback
**File:** `/src/app/api/auth/callback/route.ts:8`  
**Severity:** HIGH  
**Time:** 1h  
**Risk:** Attacker crafts link: `auth/callback?next=https://attacker.com` → phishing

**Current Code (Vulnerable):**
```typescript
export async function GET(req: Request) {
  const searchParams = new URL(req.url).searchParams;
  const next = searchParams.get('next') || '/dashboard';
  
  // No validation!
  res.redirect(next);  // ⚠️ Can redirect anywhere
}
```

**Attack:**
```
User clicks: https://leadcatcher.app/api/auth/callback?next=https://evil.com/steal-token
→ Redirects to attacker site
→ Attacker harvests auth token from URL
```

**Fix:**

```typescript
function isValidRedirectUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    const allowed = [
      new URL('https://leadcatcher.app').host,
      new URL('https://www.leadcatcher.app').host,
      // Add localhost for dev
      'localhost:3000'
    ];
    
    return allowed.includes(parsed.host);
  } catch {
    return false;
  }
}

export async function GET(req: Request) {
  const searchParams = new URL(req.url).searchParams;
  let next = searchParams.get('next') || '/dashboard';
  
  // ✓ Validate redirect URL
  if (!isValidRedirectUrl(next)) {
    next = '/dashboard'; // Fallback to safe default
  }
  
  res.redirect(next);
}
```

**Verification:**
```typescript
expect(isValidRedirectUrl('https://leadcatcher.app/dashboard')).toBe(true);
expect(isValidRedirectUrl('https://evil.com')).toBe(false);
expect(isValidRedirectUrl('/dashboard')).toBe(false); // Relative URLs not safe
```

**Deliverable:** Unit test proving only whitelisted domains allowed

---

#### SS-005: Settings Page Crash (fetchSettings Before Declaration)
**File:** `/src/app/dashboard/settings/page.tsx:38`  
**Severity:** HIGH  
**Time:** 15m  
**Risk:** Settings page crashes on load → 500 error → user can't access settings

**Current Code (Broken):**
```typescript
// Line 30
export default function SettingsPage() {
  useEffect(() => {
    // Line 36
    const settings = fetchSettings(); // ⚠️ Calling undefined function
  }, []);
  
  // Line 50
  async function fetchSettings() { // Declared AFTER useEffect
    // ...
  }
  
  return <div>...</div>;
}
```

**Why It Fails:**
- JavaScript hoisting doesn't hoist function declarations inside component bodies
- `fetchSettings` is undefined when `useEffect` runs
- Throws error: `ReferenceError: fetchSettings is not defined`

**Fix:**

```typescript
export default function SettingsPage() {
  // ✓ Declare fetchSettings BEFORE useEffect
  async function fetchSettings() {
    const res = await fetch('/api/settings');
    return res.json();
  }
  
  // Now useEffect can call it
  useEffect(() => {
    const settings = fetchSettings();
  }, []);
  
  return <div>...</div>;
}
```

**Or use separate file:**
```typescript
// utils/settings.ts
export async function fetchSettings() {
  const res = await fetch('/api/settings');
  return res.json();
}

// page.tsx
import { fetchSettings } from '@/utils/settings';

export default function SettingsPage() {
  useEffect(() => {
    const settings = fetchSettings(); // ✓ Imported, safely defined
  }, []);
}
```

**Verification:**
```bash
npm run build
# Should compile without errors
```

**Deliverable:** Settings page loads without error

---

## Deployment Sequence

### Phase 1: Iron Secretary V2 (2h 15m)
1. **SS-002:** Remove Gemini API key from vite.config.ts (15m)
2. **SS-001:** Set RLS policies on Supabase (2h)
   - Test with non-owner user
   - Verify zero data leakage

**Go/No-Go:** All voice_logs, workspace_tasks, night_shift_tasks protected with RLS

---

### Phase 2: LeadCatcher (2h 15m)
1. **SS-005:** Move fetchSettings before useEffect (15m)
   - Verify settings page loads
2. **SS-003:** Add Twilio signature validation (1h)
   - Unit test included
3. **SS-004:** Add redirect URL whitelist (1h)
   - Unit test included

**Go/No-Go:** All auth & webhook endpoints validated

---

### Phase 3: ReviewGuard (0h — No blocking issues)
- Already audit-clean
- Minor: Run `npm run lint:fix` when convenient

---

## Testing Checklist

### Iron Secretary V2
- [ ] Non-owner user cannot read voice_logs
- [ ] Non-owner user cannot read workspace_tasks
- [ ] Non-owner user cannot read night_shift_tasks
- [ ] Gemini API key NOT in dist/ bundle
- [ ] Backend proxy for Gemini API working

### LeadCatcher
- [ ] Settings page loads without errors
- [ ] Twilio webhook rejects invalid signatures
- [ ] Twilio webhook accepts valid signatures
- [ ] Auth callback rejects external redirect URLs
- [ ] Auth callback accepts whitelisted URLs

### ReviewGuard
- [ ] `npm run lint:fix` passes (0 errors)
- [ ] `.env.example` present and documented

---

## Rollback Plan

If any fix causes issues:

1. **Revert Git commit:** `git revert <commit-hash>`
2. **Restart service:** `systemctl restart leadcatcher` (or equivalent)
3. **Alert Shuki:** Immediate Telegram notification

All fixes are backward-compatible (no data migration required).

---

## Cost-Benefit Analysis

| Fix | Cost | Benefit | ROI |
|---|---|---|---|
| SS-001 (RLS) | 2h | Prevents unauthorized data access to 3 tables | 99% (prevents breach) |
| SS-002 (API key) | 15m | Prevents credential theft | 95% (limits blast radius) |
| SS-003 (Webhook sig) | 1h | Prevents fake transcription injection | 90% (stops spoofing) |
| SS-004 (Redirect) | 1h | Prevents phishing attacks | 85% (stops open redirect) |
| SS-005 (Settings) | 15m | Fixes 500 error on settings page | 75% (UX improvement) |
| **TOTAL** | **4.75h** | **Zero-day protection** | **~90% (avg)** |

**Recommendation:** Deploy all 5 fixes as a batch (Friday afternoon, 3-4h sprint).

---

## Sign-Off Checklist

- [ ] All 5 fixes reviewed by Shuki
- [ ] Testing completed on staging
- [ ] Rollback plan documented
- [ ] Team notified of deployment window
- [ ] Post-deployment verification completed
- [ ] Fixes committed to Git with security tags

---

## Next Steps

1. **Shuki approves fix sequence** (CEO decision)
2. **Jay implements fixes** in parallel (4.75h total)
3. **Stage testing** (30m)
4. **Production deployment** (15m)
5. **Monitoring** (24h post-deploy)

**Ready to proceed?** ✅
