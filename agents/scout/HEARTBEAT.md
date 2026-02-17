# HEARTBEAT.md — Scout Health Check

Run these checks periodically (every 1-2 hours during active hours).

## 1. Gateway Status
```bash
openclaw gateway status
```
**Expect:** Runtime running, RPC probe ok.

## 2. Pending Search Requests
- Check Discord #all-agents for parts requests
- Check if any searches are queued or stale

## 3. API Health
- eBay API: verify credentials still valid
- Web search: verify Brave API responding

## 4. Cost Check
- Review today's spend in `memory/cost_ledger.jsonl`
- **Alert if:** daily spend exceeds $4.00 (80% of $5 cap)

## 5. Search Log
- Check `memory/scout-searches.jsonl` for any failed searches that need retry
- Flag parts that have been searched 3+ times without a good result

## Active Hours
- 8:00 AM - 8:00 PM ET (shop hours + buffer)
- Outside active hours: silent unless urgent request comes in
