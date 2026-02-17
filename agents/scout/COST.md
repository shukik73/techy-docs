# COST.md — Scout Budget Rules

## Limits
- **Per-search cap:** $0.50 (Scout tasks are simple lookups)
- **Daily cap:** $5.00
- **Monthly cap:** $50.00 (allocated from $150/mo total agent ops)

## Model Strategy
- Use cheapest available model for searches (Gemini Flash, Haiku, GPT-mini)
- Only escalate to Sonnet/Opus if search requires complex reasoning (rare)
- Batch searches when possible (multiple parts in one query)

## Logging
- Log all costs to `memory/cost_ledger.jsonl`
- Format: `{"date":"YYYY-MM-DD","task":"parts search","model":"...","tokens_in":N,"tokens_out":N,"cost":0.XX}`

## Rules
- If a single search exceeds $0.50, stop and flag to Kai
- If daily cap is hit, pause non-urgent searches until next day
- Web searches (Brave API) are free — prefer them over LLM reasoning
- eBay API calls are free — use liberally
