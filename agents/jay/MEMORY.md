# MEMORY.md — Jay's Persistent Memory

## Owner
- **Name:** Shuki (Josh Kol)
- **Business:** DrPhoneFix & Repair Miramar — KYS Services LLC
- **Location:** Miramar, FL
- **Telegram ID:** 6103393903

## The Four Businesses
1. **Techy Miramar** — Electronics repair shop. Revenue core. Phones, laptops, PCs, game consoles.
2. **ReviewGuard** — Reputation management SaaS (GetReviewGuard.com)
3. **LeadCatcher** — Lead gen tool (LeadCatcher.app) — Twilio approval still pending
4. **Iron Secretary** — AI call analysis + PartsScout (IronSecretary.com)

## Target
- **$250K profit for 2026**
- Repair Shop Filter: if it doesn't save a technician time or drive revenue, deprioritize

## Chain of Command
- Shuki → Kai (Chief of Staff, strategic) → Jay (Squad Lead, operational) → Worker agents
- Kai runs on Claude Opus. Jay runs on Claude Sonnet (cost-efficient).
- Squad HQ: Shared Telegram group (Shuki + Kai + Jay)

## Infrastructure
- **GitHub:** `shukik73/techy-docs` (workspace), `shukik73/mission-control` (dashboard + agent prompts)
- **Mission Control:** `https://dashboard-ten-phi-45.vercel.app/`
- **n8n:** `https://n8n.srv1155599.hstgr.cloud` (API key in env var `N8N_API_KEY`)
- **OpenClaw workspace:** `~/.openclaw/workspace`
- **Jay's config:** `~/.openclaw/workspace/agents/jay/`

## Worker Agents (Planned)
Prompts exist in mission-control repo, not yet deployed:
- **Scout** — Parts sourcing (eBay, Amazon, FB Marketplace, OfferUp, Craigslist)
- **Pixel** — Visual/content
- **Buzz** — Marketing/social
- **Midas** — Revenue/pricing
- **Pluto** — Analytics
- **Emilio** — Email/outreach
- **Secretary** — Call analysis (Iron Secretary)

## Parts Sourcing Context
- **Phone screens:** Already sourced from Mobile Sentrix & Injured Gadgets — no agent needed
- **Laptop/PC parts:** The pain point. Scout agent handles this.
- **Total parts spend:** ~$67K/year ($22.6K eBay + $44.8K Amazon)
- **MacBook screens:** 32% of Amazon spend ($14K). Seven Puppy is dominant supplier.
- **SSDs:** TEAMGROUP AX2 is the workhorse brand
- **Full analyses:** `reports/ebay-purchase-analysis.md`, `reports/amazon-purchase-analysis.md`
- **Scout spec:** `reports/parts-scout-spec.md`

## Key Reports (in workspace/reports/)
- `ebay-purchase-analysis.md` — 237 orders, $22.6K, category breakdown + seller data
- `amazon-purchase-analysis.md` — 605 items, $44.8K, MacBook screen deep dive, search templates
- `parts-scout-spec.md` — Scout agent specification with Phase 2 Lister agent
- `jay-setup-guide.md` — Jay's own setup documentation
- `mission-control-setup.md` — Dashboard activation plan
- `local-intelligence-2026.md` — Local LLM deployment research (parked for Q2+)

## Open Priorities (as of 2026-02-15)
1. Iron Secretary security blockers — SS-001 (missing RLS) + SS-002 (API key exposed). CRITICAL.
2. Reddit threads — 3 active threads re: AI call solutions. Engage for Iron Secretary.
3. Missed Call Calculator landing page — high-conversion play
4. Scout agent deployment — first worker agent under Jay
5. RG cold email setup — burner domain + Instantly.ai
6. Connect Mission Control to OpenClaw gateway

## Blockers
- Iron Secretary security fixes block all outreach/onboarding
- Twilio approval pending for LeadCatcher
- No browser on MacBook (Chrome/Brave not installed) — limits browser automation

## Trust Level
- **Current: T1 (Execute)** — assigned by Kai on 2026-02-15
- T2 requires 1 week clean execution
- T3 requires demonstrated cross-business judgment

## Budget
- $150/mo total agent ops
- $2 per-task cap (escalate to Kai above this)
- Full details: `agents/jay/COST.md`
