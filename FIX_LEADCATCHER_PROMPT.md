# Claude Code Prompt: Fix LeadCatcher Security Blockers

Copy this prompt and paste into Claude Code. It will generate complete fixes for all 3 critical issues.

---

## MISSION: Fix 3 Critical Security Blockers in LeadCatcher

**Context:** LeadCatcher is a Twilio-based lead capture + nurture automation app. It has 3 critical security issues:

1. **Missing Twilio webhook signature validation** — allows fake transcription injection
2. **Open redirect in auth callback** — allows phishing attacks
3. **Settings page crash** — fetchSettings called before declaration

**Deliverables:**
- Fixed webhook route with signature validation
- Fixed auth callback with URL whitelist
- Fixed settings page with correct function ordering
- Unit tests for each fix

---

## BLOCKER 1: Missing Twilio Webhook Signature Validation

**File:** `/src/app/api/webhooks/twilio/transcription/route.ts`  
**Severity:** HIGH  
**Risk:** Attackers send fake transcriptions → auto-reply sent to innocent customers → reputation damage  
**Fix Time:** 1 hour

### Current Problem

```typescript
export async function POST(req: Request) {
  const body = await req.json();
  
  // ❌ NO SIGNATURE VALIDATION
  const { transcription, call_sid } = body;
  
  // Directly processes untrusted data
  await saveTranscription(call_sid, transcription);
  return new Response(JSON.stringify({ success: true }));
}
```

### Attack Scenario

```bash
curl -X POST https://leadcatcher.app/api/webhooks/twilio/transcription \
  -H "Content-Type: application/json" \
  -d '{
    "transcription": "SPAM MESSAGE TO DAMAGE REPUTATION",
    "call_sid": "legitimate-sid"
  }'

# Result: Fake transcription saved + auto-reply sent
```

### Solution

**Step 1: Create signature validation utility**

Create: `src/lib/twilio-validator.ts`

```typescript
import crypto from 'crypto';

export function validateTwilioSignature(
  token: string,
  signature: string,
  url: string,
  params: Record<string, any>
): boolean {
  // Reconstruct the request as Twilio sends it
  const data = url + new URLSearchParams(params).toString();
  
  // Generate HMAC-SHA1 hash
  const hash = crypto
    .createHmac('sha1', token)
    .update(data)
    .digest('base64');
  
  // Compare with provided signature (constant-time comparison)
  return crypto.timingSafeEqual(
    Buffer.from(hash),
    Buffer.from(signature)
  );
}
```

**Step 2: Update webhook route**

```typescript
import { validateTwilioSignature } from '@/lib/twilio-validator';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  try {
    // Get Twilio signature from headers
    const signature = req.headers.get('x-twilio-signature');
    if (!signature) {
      console.warn('Missing Twilio signature header');
      return NextResponse.json(
        { error: 'Missing signature' },
        { status: 403 }
      );
    }

    // Get auth token from environment
    const twilioAuthToken = process.env.TWILIO_AUTH_TOKEN;
    if (!twilioAuthToken) {
      console.error('TWILIO_AUTH_TOKEN not configured');
      return NextResponse.json(
        { error: 'Webhook validation not configured' },
        { status: 500 }
      );
    }

    // Parse request body
    const body = await req.json();

    // Reconstruct full URL (required by Twilio validation)
    const protocol = req.headers.get('x-forwarded-proto') || 'https';
    const host = req.headers.get('host');
    const url = `${protocol}://${host}${req.nextUrl.pathname}`;

    // ✓ Validate signature
    const isValid = validateTwilioSignature(
      twilioAuthToken,
      signature,
      url,
      body
    );

    if (!isValid) {
      console.warn('Invalid Twilio signature for URL:', url);
      return NextResponse.json(
        { error: 'Invalid signature' },
        { status: 403 }
      );
    }

    // ✓ Signature valid, process transcription
    const { transcription, call_sid, from } = body;

    if (!transcription || !call_sid) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Safe to save (signature validated)
    await saveTranscription({
      call_sid,
      transcription,
      from,
      validated_at: new Date().toISOString()
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('[WEBHOOK] Error:', error);
    return NextResponse.json(
      { error: 'Failed to process webhook' },
      { status: 500 }
    );
  }
}
```

### Unit Test

Create: `src/lib/__tests__/twilio-validator.test.ts`

```typescript
import { validateTwilioSignature } from '../twilio-validator';

describe('validateTwilioSignature', () => {
  const token = 'test-auth-token';
  const url = 'https://example.com/api/webhooks/twilio/transcription';
  const params = {
    CallSid: 'CA1234567890',
    Transcription: 'Hello world'
  };

  it('should validate correct signature', () => {
    const crypto = require('crypto');
    const data = url + new URLSearchParams(params).toString();
    const validSignature = crypto
      .createHmac('sha1', token)
      .update(data)
      .digest('base64');

    const result = validateTwilioSignature(token, validSignature, url, params);
    expect(result).toBe(true);
  });

  it('should reject invalid signature', () => {
    const invalidSignature = 'invalid-signature';
    const result = validateTwilioSignature(token, invalidSignature, url, params);
    expect(result).toBe(false);
  });

  it('should reject tampered params', () => {
    const crypto = require('crypto');
    const data = url + new URLSearchParams(params).toString();
    const validSignature = crypto
      .createHmac('sha1', token)
      .update(data)
      .digest('base64');

    // Try with different params
    const tamperedParams = { ...params, Transcription: 'Malicious' };
    const result = validateTwilioSignature(token, validSignature, url, tamperedParams);
    expect(result).toBe(false);
  });
});
```

---

## BLOCKER 2: Open Redirect in Auth Callback

**File:** `/src/app/api/auth/callback/route.ts`  
**Severity:** HIGH  
**Risk:** Attacker crafts link → redirects to phishing site → steals credentials  
**Fix Time:** 1 hour

### Current Problem

```typescript
export async function GET(req: Request) {
  const url = new URL(req.url);
  const next = url.searchParams.get('next') || '/dashboard';
  
  // ❌ NO VALIDATION
  res.redirect(next); // Can redirect anywhere
}
```

### Attack Scenario

```
User clicks: https://leadcatcher.app/api/auth/callback?next=https://attacker.com/steal
→ Redirects to attacker site
→ Attacker harvests auth token / credentials
```

### Solution

**Create whitelist validator**

Create: `src/lib/redirect-validator.ts`

```typescript
export function isValidRedirectUrl(
  redirectUrl: string,
  allowedHosts: string[] = []
): boolean {
  // Default allowed hosts
  const hosts = [
    'leadcatcher.app',
    'www.leadcatcher.app',
    'localhost:3000',
    'localhost:3001',
    ...allowedHosts
  ];

  try {
    // Must be absolute URL (not relative)
    const parsed = new URL(redirectUrl);

    // Host must be in whitelist
    if (!hosts.includes(parsed.host)) {
      return false;
    }

    // Must use HTTPS (except localhost for dev)
    if (parsed.protocol !== 'https:' && !parsed.host.startsWith('localhost')) {
      return false;
    }

    // Must not have dangerous protocols
    if (parsed.protocol.includes('javascript:') || parsed.protocol.includes('data:')) {
      return false;
    }

    return true;
  } catch {
    // Invalid URL format
    return false;
  }
}
```

**Update auth callback route**

```typescript
import { isValidRedirectUrl } from '@/lib/redirect-validator';
import { NextRequest, NextResponse } from 'next/server';

export async function GET(req: NextRequest) {
  try {
    const searchParams = new URL(req.url).searchParams;
    let next = searchParams.get('next') || '/dashboard';

    // ✓ Validate redirect URL
    if (!isValidRedirectUrl(next)) {
      console.warn('Invalid redirect URL attempted:', next);
      next = '/dashboard'; // Safe fallback
    }

    // Additional check: only allow relative paths for safety
    if (!next.startsWith('/')) {
      next = '/dashboard';
    }

    // Complete auth flow here (OAuth, session, etc.)
    // ...

    return NextResponse.redirect(next);
  } catch (error) {
    console.error('[AUTH] Callback error:', error);
    return NextResponse.redirect('/dashboard');
  }
}
```

### Unit Test

Create: `src/lib/__tests__/redirect-validator.test.ts`

```typescript
import { isValidRedirectUrl } from '../redirect-validator';

describe('isValidRedirectUrl', () => {
  it('should allow valid internal URLs', () => {
    expect(isValidRedirectUrl('https://leadcatcher.app/dashboard')).toBe(true);
    expect(isValidRedirectUrl('https://www.leadcatcher.app/settings')).toBe(true);
    expect(isValidRedirectUrl('http://localhost:3000/dashboard')).toBe(true);
  });

  it('should reject external URLs', () => {
    expect(isValidRedirectUrl('https://evil.com')).toBe(false);
    expect(isValidRedirectUrl('https://attacker.com/steal')).toBe(false);
  });

  it('should reject dangerous protocols', () => {
    expect(isValidRedirectUrl('javascript:alert("xss")')).toBe(false);
    expect(isValidRedirectUrl('data:text/html,<script>alert(1)</script>')).toBe(false);
  });

  it('should reject malformed URLs', () => {
    expect(isValidRedirectUrl('not-a-url')).toBe(false);
    expect(isValidRedirectUrl('//evil.com')).toBe(false);
  });

  it('should allow relative paths', () => {
    // This is handled separately (add .startsWith('/') check in route)
    // Make sure the route validates this
  });
});
```

---

## BLOCKER 3: Settings Page Crash (fetchSettings Before Declaration)

**File:** `/src/app/dashboard/settings/page.tsx`  
**Severity:** HIGH  
**Risk:** Settings page crashes on load → 500 error → user can't access settings  
**Fix Time:** 15 minutes

### Current Problem

```typescript
export default function SettingsPage() {
  useEffect(() => {
    // ❌ Calling function before it's declared
    const settings = fetchSettings();
  }, []);

  // Function declared AFTER useEffect (too late)
  async function fetchSettings() {
    const res = await fetch('/api/settings');
    return res.json();
  }

  return <div>Settings</div>;
}
```

### Why It Fails

- JavaScript hoisting doesn't hoist function declarations inside component bodies
- `fetchSettings` is undefined when `useEffect` runs
- Throws error: `ReferenceError: fetchSettings is not defined`

### Solution: Declare Before useEffect

**Option A: Extract to separate file (recommended)**

Create: `src/lib/settings.ts`

```typescript
export async function fetchSettings() {
  const response = await fetch('/api/settings');
  if (!response.ok) {
    throw new Error('Failed to fetch settings');
  }
  return response.json();
}

export async function updateSettings(data: Record<string, any>) {
  const response = await fetch('/api/settings', {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  if (!response.ok) {
    throw new Error('Failed to update settings');
  }
  return response.json();
}
```

Update page:

```typescript
'use client';

import { fetchSettings, updateSettings } from '@/lib/settings';
import { useEffect, useState } from 'react';

export default function SettingsPage() {
  const [settings, setSettings] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // ✓ fetchSettings is imported (safe)
    fetchSettings()
      .then(setSettings)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h1>Settings</h1>
      {/* Render settings */}
    </div>
  );
}
```

**Option B: Declare before useEffect (inline)**

```typescript
export default function SettingsPage() {
  // ✓ Declare function BEFORE useEffect
  async function fetchSettings() {
    const res = await fetch('/api/settings');
    return res.json();
  }

  // Now useEffect can call it
  useEffect(() => {
    const settings = fetchSettings();
  }, []);

  return <div>Settings</div>;
}
```

### Verification

```bash
npm run build
# Should compile without errors
# No "ReferenceError" in runtime console
```

---

## TESTING CHECKLIST

- [ ] Twilio webhook validates signature correctly
- [ ] Webhook rejects invalid signatures (403)
- [ ] Webhook rejects missing signature header (403)
- [ ] Valid Twilio requests pass validation
- [ ] Unit tests for signature validation pass
- [ ] Auth callback rejects external URLs (redirects to /dashboard)
- [ ] Auth callback accepts only whitelisted domains
- [ ] Unit tests for URL validation pass
- [ ] Settings page loads without errors
- [ ] Settings page fetches data on mount
- [ ] No console errors in DevTools

---

## PRIORITY

**Deploy in order:**
1. Signature validation (blocks injection attacks)
2. URL whitelist (blocks phishing)
3. Settings fix (fixes crash)

**Combined fix time:** ~2.5 hours  
**ROI:** Prevents webhook spoofing + phishing + UX crashes
