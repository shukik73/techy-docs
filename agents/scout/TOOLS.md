# TOOLS.md — Scout Tools & Resources

## APIs
- **eBay API (Production)**
  - Credentials stored in environment variables (EBAY_APP_ID, EBAY_DEV_ID, EBAY_CERT_ID)
  - Use: Finding API for parts search, Browse API for pricing

- **Amazon**: No API yet — use web search fallback

## Web Search
- Brave Search API for quick lookups
- Search patterns:
  - `"{panel P/N}" site:ebay.com`
  - `"{model} screen replacement" site:amazon.com`
  - `"{model} for parts" site:ebay.com` (salvage check)

## Vendor Websites (scrape/fetch when needed)
- laptopscreen.com — search by panel P/N
- blisscomputers.net — search by panel P/N or laptop model
- parts-people.com — search by Dell Service Tag
- injuredgadgets.com — phone/tablet parts
- mobilesentrix.com — phone/tablet + general
- protechrestore.com — Apple parts by A-number
- techy.parts — general parts

## Reference Data
- `reports/ebay-purchase-analysis.md` — 237 orders, preferred sellers, price ranges
- `reports/amazon-purchase-analysis.md` — 605 items, MacBook screen deep dive, SSD preferences
