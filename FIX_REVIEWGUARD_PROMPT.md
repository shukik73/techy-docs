# Claude Code Prompt: Clean Up ReviewGuard (Optional Improvements)

Copy this prompt and paste into Claude Code. ReviewGuard has **no critical blockers** but these cleanups improve quality.

---

## MISSION: ReviewGuard Quality Improvements (Non-Blocking)

**Context:** ReviewGuard is PRODUCTION-READY (83/100). This prompt addresses nice-to-have improvements:

1. **Fix 16 ESLint errors** — code quality
2. **Create `.env.example`** — developer experience
3. **Document CSP migration** — future security roadmap

**Priority:** LOW (audit already passed)  
**Time to complete:** 30 minutes  
**ROI:** Code quality, faster onboarding, future-proofing

---

## IMPROVEMENT 1: Fix ESLint Errors (Low Priority)

**Command to run:**
```bash
npm run lint:fix
```

This automatically fixes most issues. Remaining manual fixes:

### Common ESLint Issues Found

**a) Unused imports**
```typescript
// Before
import { unused } from './module';

// After
// Remove unused imports entirely
```

**b) Variable naming conventions**
```typescript
// Before
const User_ID = 123;

// After
const userId = 123;
```

**c) Missing semicolons (auto-fixable)**
```bash
npm run lint:fix
```

### Manual Steps

1. Run auto-fix:
```bash
npm run lint:fix
```

2. Check remaining errors:
```bash
npm run lint
```

3. For any remaining errors, review and fix manually based on output

4. Verify no new errors introduced:
```bash
npm run typecheck
```

### Testing

```bash
npm run lint
# Should output: ✓ 0 errors found
```

---

## IMPROVEMENT 2: Create `.env.example` File

**File location:** Repository root  
**Purpose:** Document required environment variables for developers

Create: `.env.example`

```env
# ========================================
# ReviewGuard Environment Configuration
# ========================================
# Copy this file to .env.local and fill in your values

# DATABASE CONFIGURATION
DATABASE_URL=postgresql://user:password@localhost:5432/reviewguard_db

# SESSION & SECURITY
SESSION_SECRET=your-very-secure-256-bit-random-string-here
NODE_ENV=development

# OPENAI API (For AI-powered review replies)
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxx

# TWILIO (SMS & Voice)
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your-twilio-auth-token
TWILIO_PHONE_NUMBER=+1234567890

# STRIPE (Billing & Subscriptions)
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PUBLISHABLE_KEY=pk_live_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_ID_STARTER=price_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_ID_PRO=price_xxxxxxxxxxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxxx

# RESEND (Email delivery)
RESEND_API_KEY=re_xxxxxxxxxxxxxxxxxxxxx

# SENTRY (Error tracking - optional)
SENTRY_DSN=https://xxxxx@sentry.io/xxxxx
SENTRY_TRACES_SAMPLE_RATE=0.1
NODE_ENV=production

# CLOUDINARY (Image storage)
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret

# DEPLOYMENT
BASE_URL=https://getreviewguard.com
REPLIT_DEPLOYMENT=0
COOKIE_DOMAIN=.getreviewguard.com

# FEATURE FLAGS
SKIP_TWILIO_WEBHOOK_VALIDATION=false  # Only for development!

# N8N (Automation workflows)
N8N_CAMPAIGN_WEBHOOK_URL=https://your-n8n-instance.com/webhook/campaigns

# REDIS (Caching & Rate limiting)
REDIS_URL=redis://localhost:6379

# ========================================
# HOW TO USE:
# ========================================
# 1. Copy this file: cp .env.example .env.local
# 2. Fill in your values (ask team members for credentials)
# 3. Never commit .env.local to Git (it's in .gitignore)
# 4. For production, configure these in your deployment platform's secrets
#
# REQUIRED VARIABLES (must be set):
# - DATABASE_URL
# - SESSION_SECRET (generate: openssl rand -hex 32)
# - OPENAI_API_KEY
# - TWILIO_ACCOUNT_SID
# - TWILIO_AUTH_TOKEN
#
# OPTIONAL VARIABLES (have fallbacks):
# - SENTRY_DSN
# - CLOUDINARY_* (falls back to local storage)
```

### Update .gitignore

Ensure `.env.local` is never committed:

```
.env.local
.env.*.local
.env.production.local
```

### Documentation

Add to README.md:

```markdown
## Setup

1. Install dependencies:
   ```bash
   npm install
   ```

2. Configure environment:
   ```bash
   cp .env.example .env.local
   # Edit .env.local with your credentials
   ```

3. Run migrations:
   ```bash
   npm run migrate
   ```

4. Start development server:
   ```bash
   npm run dev
   ```
```

---

## IMPROVEMENT 3: Document CSP Migration Plan (Future Roadmap)

**File:** `docs/SECURITY_ROADMAP.md`

Create this document:

```markdown
# ReviewGuard Security Roadmap

## Current Status: Production-Ready ✅

ReviewGuard passes all security audits. The following improvements are planned for future releases.

---

## P1: Nonce-Based CSP Migration (Q2 2026)

**Current State:**
- CSP configured with `'unsafe-inline'` for scripts
- Required because frontend uses inline event handlers (`onclick=...`)
- This limits XSS protection

**Future Improvement:**
- Replace all `onclick="handler()"` with `addEventListener()` + data attributes
- Migrate to nonce-based CSP headers
- Eliminates `unsafe-inline` requirement
- Improves XSS protection significantly

**Work Breakdown:**
1. Audit frontend components (2h)
2. Refactor event handlers (6-8h)
3. Implement nonce generation middleware (2h)
4. Update CSP policy (1h)
5. Testing & QA (2h)

**Total Effort:** ~14 hours  
**Timeline:** Q2 2026  
**Risk Level:** Low (non-breaking refactor)

---

## P2: Additional Security Measures

- [ ] HSTS header hardening
- [ ] Subresource Integrity (SRI) for CDN resources
- [ ] Certificate Transparency monitoring
- [ ] Regular security audits (quarterly)

---

## How to Track Progress

View the [GitHub Projects](https://github.com/shukik73/reviewguard-app/projects/1) board for current work.

---

## Questions?

Contact security team or create an issue in GitHub.
```

---

## TESTING CHECKLIST

- [ ] Run `npm run lint` → 0 errors
- [ ] Run `npm run typecheck` → 0 errors  
- [ ] `.env.example` created and documented
- [ ] `.env.local` in `.gitignore`
- [ ] README updated with setup instructions
- [ ] `docs/SECURITY_ROADMAP.md` created
- [ ] All tests still pass

---

## SUMMARY

| Task | Time | Priority | ROI |
|------|------|----------|-----|
| `npm run lint:fix` | 10m | Medium | Cleaner code |
| `.env.example` | 10m | High | Better DX |
| CSP roadmap docs | 10m | Low | Future planning |
| **TOTAL** | **30m** | — | **Quality boost** |

**These are nice-to-have, not blocking.** ReviewGuard is already production-ready.

**Run when convenient:** Next dev sprint or Friday cleanup session.
