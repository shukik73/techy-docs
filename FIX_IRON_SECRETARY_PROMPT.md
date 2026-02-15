# Claude Code Prompt: Fix Iron Secretary V2 Security Blockers

Copy this prompt and paste into Claude Code. It will generate the complete fixes for both critical security issues.

---

## MISSION: Fix 2 Critical Security Blockers in Iron Secretary V2

**Context:** Iron Secretary V2 is an AI call analysis + night shift task automation app. It has 2 critical security issues that prevent production deployment:

1. **RLS (Row-Level Security) missing** on 3 Supabase tables
2. **Gemini API key exposed** in client bundle

**Deliverables:** 
- SQL migration file for RLS policies
- Updated vite.config.ts removing API key exposure
- Verification instructions

---

## BLOCKER 1: Missing RLS on Supabase Tables

**Severity:** CRITICAL  
**Risk:** Any authenticated user can read all other users' data (voice_logs, workspace_tasks, night_shift_tasks)  
**Fix Time:** 15 minutes (to implement) + 5 minutes (to test)

### What to Fix

Supabase tables created without Row-Level Security:
- `voice_logs` — Contains all call transcriptions + audio metadata (PII)
- `workspace_tasks` — Contains all AI workspace data
- `night_shift_tasks` — Contains all night shift orchestration data

### Current Problem

```sql
-- Tables exist but no RLS enabled
SELECT * FROM voice_logs; -- ANY authenticated user can read all rows
```

### Solution

Create a Supabase migration file: `supabase/migrations/[timestamp]_enable_rls.sql`

```sql
-- 1. Enable RLS on all 3 tables
ALTER TABLE voice_logs ENABLE ROW LEVEL SECURITY;
ALTER TABLE workspace_tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE night_shift_tasks ENABLE ROW LEVEL SECURITY;

-- 2. Create policies for voice_logs
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

CREATE POLICY "Users can delete own voice logs"
  ON voice_logs
  FOR DELETE
  USING (auth.uid() = user_id);

-- 3. Create policies for workspace_tasks
CREATE POLICY "Users can read own workspace tasks"
  ON workspace_tasks
  FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own workspace tasks"
  ON workspace_tasks
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own workspace tasks"
  ON workspace_tasks
  FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own workspace tasks"
  ON workspace_tasks
  FOR DELETE
  USING (auth.uid() = user_id);

-- 4. Create policies for night_shift_tasks
CREATE POLICY "Users can read own night shift tasks"
  ON night_shift_tasks
  FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own night shift tasks"
  ON night_shift_tasks
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own night shift tasks"
  ON night_shift_tasks
  FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own night shift tasks"
  ON night_shift_tasks
  FOR DELETE
  USING (auth.uid() = user_id);

-- 5. Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_voice_logs_user_id ON voice_logs(user_id);
CREATE INDEX IF NOT EXISTS idx_workspace_tasks_user_id ON workspace_tasks(user_id);
CREATE INDEX IF NOT EXISTS idx_night_shift_tasks_user_id ON night_shift_tasks(user_id);
```

### Testing Instructions

After applying migration in Supabase dashboard:

```sql
-- Test 1: Log in as user A (uuid: 12345...)
-- Should return data only for user A
SELECT * FROM voice_logs; -- Returns 5 rows (user A's data only)

-- Test 2: Try to query user B's data explicitly
-- Should be blocked by RLS
SELECT * FROM voice_logs WHERE user_id = 'different-uuid'; -- Returns 0 rows (blocked)

-- Test 3: Log in as user B
-- Should return data only for user B
SELECT * FROM voice_logs; -- Returns 3 rows (user B's data only)
```

### What to Deliver

1. SQL migration file with complete RLS policy setup
2. Comment in migration explaining each policy
3. Testing checklist showing RLS prevents cross-user access

---

## BLOCKER 2: Gemini API Key Exposed in Client Bundle

**Severity:** CRITICAL  
**Risk:** API key extractable from browser → attackers can call Gemini API on your dime  
**Fix Time:** 20 minutes

### Current Problem

**File:** `vite.config.ts`

```typescript
export default defineConfig({
  define: {
    'process.env.VITE_GEMINI_API_KEY': JSON.stringify(process.env.GEMINI_API_KEY)
  }
})
```

**Why it's dangerous:**
- Key is injected into client JavaScript
- Visible in `dist/assets/*.js` after build
- Attackers extract key from DevTools or bundled JavaScript
- No rate limiting on client-side API calls
- Bill is charged to your account indefinitely

### Solution: Backend Proxy Pattern

**Step 1: Remove from vite.config.ts**

```typescript
// vite.config.ts
export default defineConfig({
  // DELETE THIS:
  // 'process.env.VITE_GEMINI_API_KEY': JSON.stringify(process.env.GEMINI_API_KEY)
  
  // Keep only safe variables
  define: {
    'process.env.VITE_APP_NAME': JSON.stringify(process.env.VITE_APP_NAME || 'Iron Secretary')
  }
})
```

**Step 2: Create backend API endpoint**

Create new file: `src/pages/api/ai/chat.ts`

```typescript
import { GoogleGenerativeAI } from '@google/generative-ai';
import { NextRequest, NextResponse } from 'next/server';

// Verify user is authenticated
export async function POST(req: NextRequest) {
  try {
    // Check authentication (add your auth check here)
    const authHeader = req.headers.get('authorization');
    if (!authHeader) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { message } = await req.json();

    if (!message) {
      return NextResponse.json(
        { error: 'Message is required' },
        { status: 400 }
      );
    }

    // ✓ API key stays on server (never exposed to client)
    const geminiKey = process.env.GEMINI_API_KEY;
    if (!geminiKey) {
      return NextResponse.json(
        { error: 'Gemini API not configured' },
        { status: 500 }
      );
    }

    const genAI = new GoogleGenerativeAI(geminiKey);
    const model = genAI.getGenerativeModel({ model: 'gemini-pro' });

    const result = await model.generateContent(message);
    const text = result.response.text();

    return NextResponse.json({ response: text });
  } catch (error) {
    console.error('AI API error:', error);
    return NextResponse.json(
      { error: 'Failed to process request' },
      { status: 500 }
    );
  }
}
```

**Step 3: Update frontend to call backend instead**

Find all places calling Gemini API directly and replace with:

```typescript
// OLD (vulnerable):
// const genAI = new GoogleGenerativeAI(process.env.VITE_GEMINI_API_KEY);
// const model = genAI.getGenerativeModel({ model: 'gemini-pro' });
// const result = await model.generateContent(message);

// NEW (secure):
async function callAI(message: string) {
  const response = await fetch('/api/ai/chat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${getAuthToken()}` // Your auth mechanism
    },
    body: JSON.stringify({ message })
  });

  const data = await response.json();
  return data.response;
}
```

### Verification

**Before fix (vulnerable):**
```bash
npm run build
# Search bundle for API key
grep -r "AIza" dist/
# Result: FOUND IN dist/assets/main.*.js ❌
```

**After fix (secure):**
```bash
npm run build
# Search bundle for API key
grep -r "AIza" dist/
# Result: NOT FOUND ✓
grep -r "GEMINI" dist/
# Result: NOT FOUND ✓
```

### What to Deliver

1. Updated `vite.config.ts` with API key reference removed
2. New backend endpoint: `src/pages/api/ai/chat.ts`
3. Updated frontend function calling `/api/ai/chat` instead of direct Gemini
4. Verification screenshot showing `grep -r "AIza" dist/` returns nothing
5. Instructions for testing: Make request to `/api/ai/chat` and confirm response works

---

## TESTING CHECKLIST

- [ ] RLS policies applied to all 3 tables
- [ ] Non-owner user cannot read any other user's voice_logs
- [ ] Non-owner user cannot read any other user's workspace_tasks
- [ ] Non-owner user cannot read any other user's night_shift_tasks
- [ ] Gemini API key NOT found in dist/ bundle
- [ ] Backend `/api/ai/chat` endpoint responds correctly
- [ ] Frontend calls `/api/ai/chat` instead of direct Gemini
- [ ] All existing AI features still work (NightShift, AIWorkspace, Voice)

---

## PRIORITY

- **RLS policies:** Apply immediately to Supabase
- **API key removal:** Deploy before next production release

**Combined fix time:** ~35 minutes
**ROI:** Prevents unauthorized data access + credential theft
