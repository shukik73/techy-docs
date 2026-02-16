# Parts Scout Agent — Spec Doc
## Status: TODO — Waiting on gaming laptop specs + Jay onboarding

### Problem
Manual parts sourcing for laptop screens, motherboards, and PC parts takes hours. Need automated multi-platform price comparison.

### Scope
**In scope:** Laptop screens, motherboards, full laptops, PC components
**Out of scope:** Phone screens (handled via Mobile Sentrix & Injured Gadgets)

### Search Logic
1. **Primary search:** By screen model number (e.g., B156HAN02.1)
2. **Fallback:** By laptop model (e.g., "Dell Latitude 5520 screen")
3. **Platforms:** Amazon → eBay → Bliss Computers (fallback)

### Pricing & Condition Matrix

| Category | Price Range | Preferred Condition |
|---|---|---|
| Windows regular screen | $50–150 | New / Open Box |
| Windows touch screen | $150–300 | New / Open Box |
| MacBook screen (entry) | $250–300 | New / Grade A / B+ |
| MacBook screen (mid) | $300–450 | New / Grade A / B+ |
| MacBook screen (high) | $450–750 | New / Grade A |
| MacBook (budget option) | varies | B or B- (if customer agrees) |

### Grade Definitions
- **New:** Factory sealed
- **Open Box:** Opened but unused
- **Grade A:** Like new, no visible defects
- **B+:** Minor cosmetic imperfections
- **B:** Noticeable cosmetic wear
- **B-:** Significant wear, functional

### Output Format
Comparison table with:
- Price
- Condition/Grade
- Direct link to listing
- Platform (Amazon/eBay/Bliss)
- Shipping estimate (if available)

### Architecture
- Dedicated gaming laptop running local agent
- eBay API (credentials already configured)
- Amazon Product Advertising API (need credentials)
- Bliss Computers (scrape or browser automation)
- Telegram interface for queries + results

### Dependencies
- [ ] Gaming laptop specs from Shuki
- [ ] Jay agent online
- [ ] Amazon API credentials
- [ ] Bliss Computers access method confirmed
- [ ] Define top 20 most-searched parts for testing

---

## Phase 2: Lister Agent — Auto-Post Listings

### Problem
Devices sitting unsold because listing takes too long (photos, descriptions, posting to multiple platforms).

### Workflow
1. Shuki sends photos + basic info (device, condition, price)
2. Photo Agent: enhances images (background removal, lighting, crop)
3. Lister Agent: generates optimized title + description per platform
4. Auto-posts to eBay + Facebook Marketplace
5. Returns confirmation with live links

### Platforms
- eBay (API — already have credentials)
- Facebook Marketplace (browser automation on dedicated laptop)

### Photo Enhancement
- Background removal (clean white/neutral)
- Brightness/contrast normalization
- Consistent crop/framing
- Optional: branded watermark

### Listing Copy
- Platform-optimized titles (eBay SEO vs FB casual)
- Condition description from grade input
- Specs auto-filled from device model
- Competitive pricing suggestion based on current market

### Dependencies
- [ ] eBay Sell API access (different from Browse API)
- [ ] FB Marketplace posting method (browser automation)
- [ ] Photo enhancement model/API selection
- [ ] Template library for common device types

---

### Priority
HIGH — Direct margin impact. Sourcing agent saves on buy side, Lister agent accelerates sell side. Together they compress the full buy→sell cycle.
