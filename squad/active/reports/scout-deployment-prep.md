# Scout Agent Deployment Prep

**Auditor:** Jay (Squad Lead)
**Date:** 2026-02-16
**Status:** READY FOR PHASE 1 DEPLOYMENT

---

## Executive Summary

Scout agent code is **production-ready** for eBay-only Phase 1. Code exists in `mission-control/dashboard/api/cron/scout.ts` (575 lines). Missing: Amazon API and Bliss Computers integration (Phase 2).

| Component | Status | Notes |
|-----------|--------|-------|
| eBay OAuth2 | ✅ Ready | Credentials via env vars |
| eBay Browse API | ✅ Ready | Search + filtering |
| Ghost Protocol | ✅ Ready | ROI, seller rating, red flags |
| Telegram alerts | ✅ Ready | Deal notifications |
| Mission Control DB | ✅ Ready | missions + scout_deals tables |
| CRON security | ✅ Ready | CRON_SECRET validation (fail-closed) |
| Amazon API | ❌ Missing | Need API credentials |
| Bliss Computers | ❌ Missing | Need scraping/automation |

**Recommendation:** Deploy Phase 1 (eBay-only) now. Add Amazon/Bliss in Phase 2.

---

## Existing Code Analysis

### Location
```
mission-control/dashboard/api/cron/scout.ts (575 lines)
```

### Core Features (Working)

1. **eBay OAuth2 Token Acquisition**
   - Client credentials flow
   - Credentials from `EBAY_APP_ID` + `EBAY_CERT_ID` env vars

2. **eBay Browse API Search**
   - Searches 10 predefined queries
   - Filters: US only, $5-$500, Used/Very Good condition
   - Excludes: water damage, iCloud locked, BIOS locked, blacklisted

3. **Ghost Protocol Filter**
   - Seller rating >= 90% with 50+ feedback
   - Shipping cost <= 30% of item price
   - ROI >= 20% (includes eBay 12.9% + payment 2.9% fees)
   - Electronics require 20% ROI, non-electronics require 100%

4. **Market Value Estimation**
   - Checks `price_benchmarks` table first
   - Falls back to keyword-based estimates (MacBook Pro: $450, iPhone 15: $500, etc.)

5. **Deal Creation**
   - Creates mission in `missions` table
   - Creates deal in `scout_deals` table
   - Priority: urgent (ROI>200% + <2h left), high (ROI>150% or <6h), normal

6. **Telegram Notifications**
   - Sends deal alert with price, ROI, profit, seller info
   - Urgent deals get special formatting
   - Logs to `telegram_notifications` table

7. **Local Pickup Detection**
   - Detects South FL cities (Miramar, Pembroke, Hollywood, etc.)
   - Estimates distance in miles

8. **Security**
   - CRON_SECRET validation (fail-closed)
   - Title sanitization (prompt injection mitigation)
   - Batch dedup checking

### Current Search Queries
```typescript
const CRON_SEARCH_QUERIES = [
  'MacBook Pro logic board',
  'iPhone screen replacement OEM',
  'laptop battery replacement',
  'iPad logic board',
  'Samsung Galaxy screen',
  'Dell laptop motherboard',
  'iPhone charging port flex',
  'MacBook keyboard replacement',
  'Apple Watch screen',
  'PS5 HDMI port repair',
];
```

---

## Gap Analysis

### What Spec Says vs What Exists

| Spec Requirement | Status | Gap |
|------------------|--------|-----|
| eBay API integration | ✅ Complete | — |
| Amazon Product API | ❌ Missing | Need credentials |
| Bliss Computers | ❌ Missing | Need scraping/automation |
| Search by screen model number | ⚠️ Partial | Generic queries, not specific model numbers |
| Search by laptop model fallback | ⚠️ Partial | Not implemented |
| Top 20 most-searched parts | ❌ Missing | Current queries are general |
| Telegram interface for queries | ✅ Complete | Notifications work |
| Comparison table output | ⚠️ Partial | Single-platform only |
| Price range validation | ✅ Complete | Ghost Protocol handles this |
| Condition/grade filtering | ✅ Complete | Used/Very Good condition |

### Missing from Spec
1. **Amazon Product Advertising API credentials**
2. **Bliss Computers access method** (scrape vs browser automation)
3. **Top 20 most-searched parts list** from Shuki
4. **Gaming laptop specs** (spec mentions dedicated laptop)

---

## Deployment Checklist

### Phase 1: eBay-Only (Ready Now)

- [ ] **Deploy dashboard to Vercel**
  ```bash
  cd mission-control/dashboard
  vercel deploy
  ```

- [ ] **Set Environment Variables in Vercel**
  ```
  SUPABASE_URL=<from Supabase dashboard>
  SUPABASE_SERVICE_KEY=<service role key>
  EBAY_APP_ID=<eBay developer credentials>
  EBAY_CERT_ID=<eBay developer credentials>
  TELEGRAM_BOT_TOKEN=<Jay bot token>
  SHUKI_TELEGRAM_ID=<Shuki's Telegram user ID>
  CRON_SECRET=<generate random 32-char string>
  ```

- [ ] **Configure Vercel Cron**
  Add to `vercel.json`:
  ```json
  {
    "crons": [
      {
        "path": "/api/cron/scout",
        "schedule": "0 */4 * * *"
      }
    ]
  }
  ```
  This runs Scout every 4 hours.

- [ ] **Create Supabase Tables** (if not exists)
  - `missions`
  - `scout_deals`
  - `scout_trends`
  - `telegram_notifications`
  - `agents`
  - `agent_activity`
  - `price_benchmarks`

- [ ] **Seed price_benchmarks table**
  Add known market values for common parts.

- [ ] **Test manually**
  ```bash
  curl -H "Authorization: Bearer $CRON_SECRET" https://your-vercel-domain/api/cron/scout
  ```

### Phase 2: Multi-Platform (Future)

- [ ] Obtain Amazon Product Advertising API credentials
- [ ] Implement Amazon search function
- [ ] Determine Bliss Computers access method
- [ ] Implement Bliss scraping/automation
- [ ] Update search queries with top 20 most-searched parts
- [ ] Add screen model number search capability

---

## Cost Estimate

### Within $50/mo Budget

| Resource | Cost | Notes |
|----------|------|-------|
| Vercel Pro | $0 | Hobby tier sufficient for cron |
| eBay API | $0 | Browse API is free |
| Telegram API | $0 | Free |
| Supabase | $0 | Already have project |
| **Total** | **$0** | ✅ Under budget |

### API Limits to Watch
- eBay Browse API: 5,000 calls/day (search)
- With 10 queries × 6 runs/day = 60 API calls — well under limit

### Token Costs (if AI processing added)
- Current code doesn't use OpenAI/Claude for processing
- Future AI-enhanced filtering: estimate ~$5/mo at current volume

---

## Activation Steps (For Kai/Shuki)

1. **Kai approves deployment** (T1 can't deploy)
2. **Shuki provides:**
   - Confirmation eBay credentials are in a password manager
   - Top 20 most-searched parts list (for Phase 2 query optimization)
   - Amazon API decision (apply for credentials or skip Phase 2)
3. **Deploy to Vercel** with env vars
4. **Run test cron** manually
5. **Monitor first 24 hours** for false positives/negatives
6. **Tune Ghost Protocol thresholds** based on results

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Bad deals pass filter | Medium | Low ($) | Ghost Protocol has 20% ROI floor |
| Missed good deals | Medium | Low ($) | Monitor scout_trends for passed deals |
| eBay rate limiting | Low | Medium | 60 calls/day << 5,000 limit |
| Telegram spam | Low | Low | Consolidate notifications |
| Cron not firing | Medium | Medium | Monitor `agents.last_heartbeat` |

---

## Recommendation

**Deploy Phase 1 immediately.** Scout is production-ready for eBay-only operation. The $50/mo budget is untouched since all APIs are free tier.

Phase 2 (Amazon + Bliss) requires:
1. Amazon API credentials (Shuki to apply)
2. Decision on Bliss Computers automation approach
3. Browser automation infrastructure (possibly n8n or dedicated VM)

---

*Report generated by Jay | Squad Lead | T1 (Execute)*
