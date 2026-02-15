# Token & Cost Optimization Guide for OpenClaw

**Date:** 2026-02-14  
**Author:** Cost Optimization Subagent  
**Budget:** $0.50/hr · $5/day · $100/mo

---

## 1. Model Pricing Reference (Feb 2026)

| Model | Input/1M | Output/1M | Cache Read/1M | Best For |
|-------|----------|-----------|---------------|----------|
| **Claude Opus 4.5** | $5.00 | $25.00 | $0.50 | Complex reasoning, strategy |
| **Claude Sonnet 4.5** | $3.00 | $15.00 | $0.30 | Balanced — agents, code |
| **Claude Haiku 4.5** | $1.00 | $5.00 | $0.10 | High-volume, triage, simple tasks |
| Claude Haiku 3.5 | $0.80 | $4.00 | $0.08 | Budget fallback |
| **GPT-4.1** | $2.00 | $8.00 | — | Code, long-context |
| GPT-4o | $2.50 | $10.00 | — | General |
| GPT-4o-mini | $0.15 | $0.60 | — | Ultra-cheap tasks |
| **Gemini 2.0 Flash** | $0.10 | $0.40 | — | Cheapest capable model |
| Gemini 2.5 Pro | $1.25 | $10.00 | — | Google ecosystem tasks |

**Key insight:** Gemini 2.0 Flash is 50-250x cheaper than Opus 4.5. GPT-4o-mini is 33-42x cheaper.

---

## 2. Model Routing Optimization

### Task → Model Mapping

| Task | Current Model | Recommended | Savings |
|------|--------------|-------------|---------|
| Interactive chat with user | Opus 4.5 | **Sonnet 4.5** (90% of tasks) | 40% |
| Morning brief (cron) | Opus 4.5 | **Sonnet 4.5** | 40% |
| Radar nightly scan | Opus 4.5 | **Sonnet 4.5** | 40% |
| Sage SEO weekly | Opus 4.5 | **Haiku 4.5** | 80% |
| Heartbeat checks | Opus 4.5 | **Eliminate or use Haiku** | 95%+ |
| Classification/triage | Opus 4.5 | **Haiku 4.5 or GPT-4o-mini** | 80-97% |
| Web search summarization | Opus 4.5 | **Haiku 4.5** | 80% |
| Subagent: research tasks | Opus 4.5 | **Sonnet 4.5** | 40% |
| Subagent: simple file ops | Opus 4.5 | **Haiku 4.5** | 80% |

### Rule of Thumb
- **Opus**: Only for strategic decisions, complex multi-step reasoning, or when user explicitly needs peak quality
- **Sonnet**: Default for everything else — chat, agents, code, reports
- **Haiku**: Cron jobs, data extraction, classification, formatting, SEO checks

---

## 3. Prompt Engineering for Token Reduction

### 3.1 System Prompt Compression
Your current cron job prompts are verbose. Example — the Radar nightly scan payload is ~250 words. This gets sent every night as input tokens.

**Technique: Compress cron prompts by 50-70%**

Before (Radar, ~250 words → ~330 tokens):
```
You are Radar, a competitive intelligence and opportunity scout. Run a nightly scan for revenue opportunities across Shuki's four businesses...
[long description of each business, platforms to search, output format]
```

After (~100 words → ~130 tokens):
```
Radar: Nightly competitive intel scan.
Businesses: Techy Miramar (phone/electronics repair, Miramar FL), ReviewGuard (reputation SaaS), LeadCatcher (lead gen), Iron Secretary (AI call analysis).
Search: X/Twitter + Reddit + ProductHunt for people needing repair/reputation/leads/automation.
Per opportunity: link, relevance, action, revenue impact (L/M/H).
Full report → /reports/radar-nightly-YYYY-MM-DD.md
Telegram: top 3 opportunities only.
```

**Savings:** ~200 tokens/run × $5/1M (Opus input) = negligible per-run, but switching model matters more.

### 3.2 Context Window Management
- **Don't load INTELLECT.md, EVOLVE.md, AGENTS.md** in Phase 0 (your RUNTIME.md already says this — saving ~3000 tokens/session)
- For cron jobs using `sessionTarget: "isolated"`, ensure minimal system prompt injection
- Strip workspace file injection for cron jobs that don't need it

### 3.3 Shorter Output Instructions
Add to prompts: `Be concise. No preamble.` — saves 100-500 output tokens per response. At $25/1M output (Opus), 500 tokens = $0.0125. Over 20 interactions/day = $0.25/day saved.

---

## 4. OpenClaw-Specific Optimizations

Based on COST.md and RUNTIME.md:

### 4.1 Already Configured (Good)
- Budget enforcement in orchestration layer (not LLM)
- Per-trigger cost cap of $0.20
- Kill switch mechanism
- Cost ledger tracking

### 4.2 Recommended Changes

**a) Switch default model from Opus to Sonnet 4.5**
This is the single highest-impact change. Opus should be on-demand only.

**b) Heartbeat frequency**
Currently hourly. Each heartbeat = LLM call even if nothing to do.
- **Reduce to every 4 hours** (6/day instead of 24/day)
- Or make heartbeat a pure code check (no LLM) that only calls LLM when a trigger fires

**c) Session compaction**
For long interactive sessions, context grows. OpenClaw should compact/summarize older messages. If this isn't built-in, implement a rolling window: keep last 10 messages + a summary of older ones.

**d) Thinking budget**
If using extended thinking (Claude), set `thinking.budget_tokens` to minimum needed. Default can be wasteful.

---

## 5. Subagent Efficiency

### Current Problem
Subagents inherit the default model (Opus 4.5) and get large context injections (AGENTS.md, TOOLS.md, workspace files, subagent context boilerplate).

### Optimizations

| Change | Token Savings |
|--------|--------------|
| Route subagents to Sonnet 4.5 by default | 40% cost reduction |
| Use Haiku 4.5 for simple subagents (file ops, formatting) | 80% cost reduction |
| Trim subagent system prompt — remove TOOLS.md, AGENTS.md injection for tasks that don't need them | ~1000-2000 input tokens/spawn |
| Set explicit `max_tokens` per subagent (e.g., 2000 for simple tasks, 4000 for reports) | Prevents verbose output |
| Avoid spawning subagents for tasks the main agent can do in one turn | Eliminates overhead entirely |

### Subagent Prompt Template (Lean)
```
Task: [one sentence]
Output: [file path or format]
Constraints: Be concise. Max 1500 words.
```

---

## 6. Cron Job Optimization

### Current Jobs & Estimated Cost Per Run

Assuming ~2K input tokens (prompt + system) + ~2K output tokens per cron job on Opus 4.5:
- Input: 2K × $5/1M = $0.01
- Output: 2K × $25/1M = $0.05
- **Per cron run: ~$0.06 on Opus**

But cron jobs use tool calls (web_search, file reads/writes), which add turns. Realistic estimate with 5-10 tool-use rounds:
- **Per cron run: ~$0.15-0.30 on Opus** (cumulative context grows each turn)

| Job | Frequency | Model | Est. Cost/Run | Monthly Cost |
|-----|-----------|-------|---------------|-------------|
| Morning Brief | Daily | Opus | $0.25 | $7.50 |
| Radar Scan | Daily | Opus | $0.30 | $9.00 |
| Sage SEO | Weekly | Opus | $0.20 | $0.80 |
| Heartbeat | 24x/day | Opus | $0.06 | $43.20 |
| **Cron Total** | | | | **$60.50/mo** |

### Optimized Schedule

| Job | Frequency | Model | Est. Cost/Run | Monthly Cost |
|-----|-----------|-------|---------------|-------------|
| Morning Brief | Daily | **Sonnet 4.5** | $0.12 | $3.60 |
| Radar Scan | **3x/week** (M/W/F) | **Sonnet 4.5** | $0.15 | $1.95 |
| Sage SEO | Weekly | **Haiku 4.5** | $0.04 | $0.16 |
| Heartbeat | **Every 4hr** (6/day) | **Haiku 4.5** | $0.01 | $1.80 |
| **Cron Total** | | | | **$7.51/mo** |

**Savings: ~$53/mo from cron optimization alone (87% reduction)**

---

## 7. Caching and Deduplication

### 7.1 Anthropic Prompt Caching
Claude supports prompt caching — repeated system prompts are cached and charged at cache read rate ($0.50/1M for Opus vs $5.00/1M input). That's **90% savings on system prompt tokens**.

**Action:** Ensure system prompts are placed first and marked as cacheable. For cron jobs running the same prompt daily, this is automatic after first call within the cache TTL (5 minutes — so only helps within a session, not across daily runs).

**Limitation:** Cache TTL is 5 minutes. For cron jobs hours apart, caching doesn't help. It helps most for:
- Multi-turn conversations (system prompt cached across turns) ← **big win**
- Rapid successive subagent spawns

### 7.2 Avoid Redundant Web Fetches
- Radar scan searches similar queries nightly. Cache results locally and only re-fetch if >24h old.
- Morning brief fetches weather + news — consider caching partial results.

### 7.3 Batch API
Anthropic offers 50% discount for batch (non-real-time) requests. Cron jobs are perfect candidates since they don't need instant responses.
- **Batch pricing (Opus 4.5):** $2.50 input / $12.50 output per 1M tokens
- **Batch pricing (Sonnet 4.5):** $1.50 input / $7.50 output per 1M tokens

If OpenClaw supports batch API routing for cron jobs, this halves cron costs again.

---

## 8. Budget Math

### Current Setup (All Opus, No Optimization)

| Category | Est. Monthly Cost |
|----------|------------------|
| Cron jobs (as calculated above) | $60.50 |
| Interactive chat (~10 exchanges/day, ~$0.50/day) | $15.00 |
| Subagent spawns (~2/day, ~$0.30 each) | $18.00 |
| **Total** | **$93.50/mo** |

This is near the $100/mo ceiling with zero headroom.

### Optimized Setup

| Category | Changes | Est. Monthly Cost |
|----------|---------|------------------|
| Cron jobs | Sonnet/Haiku + reduced frequency | $7.51 |
| Interactive chat | Sonnet default, Opus on-demand | $9.00 |
| Subagent spawns | Sonnet/Haiku per task | $7.20 |
| **Total** | | **$23.71/mo** |

**Total savings: ~$70/mo (75% reduction)**  
**Remaining headroom: ~$76/mo for growth, new agents, or burst usage**

---

## 9. Recommended Configuration Changes

### Priority 1: Switch Default Model (Immediate, Biggest Impact)
```
# Change OpenClaw default model from Opus to Sonnet 4.5
default_model: anthropic/claude-sonnet-4-5
# Keep Opus available for explicit escalation only
```

### Priority 2: Optimize Cron Jobs
1. **Heartbeat:** Change from `0 * * * *` to `0 */4 * * *` (every 4 hours)
2. **Radar scan:** Change from daily to 3x/week: `0 1 * * 1,3,5`
3. **All cron jobs:** Add model override to use Sonnet or Haiku
4. **Compress all cron prompt payloads** by 50%+ (see Section 3.1)

### Priority 3: Subagent Model Routing
- Default subagent model: Sonnet 4.5
- For file-only / formatting tasks: Haiku 4.5
- Set `max_tokens` explicitly per subagent task

### Priority 4: Prompt Hygiene
- Add `Be concise. No preamble.` to all system prompts
- Remove AGENTS.md, INTELLECT.md, EVOLVE.md from Phase 0 context loading (per RUNTIME.md recommendation)
- Trim TOOLS.md injection for sessions that don't need tool guidance

### Priority 5: Caching
- Ensure prompt caching is enabled for multi-turn conversations
- Investigate batch API support in OpenClaw for cron jobs
- Cache web search results locally for recurring queries

---

## Summary: Top 5 Actions by Impact

| # | Action | Monthly Savings | Effort |
|---|--------|----------------|--------|
| 1 | Switch default model to Sonnet 4.5 | ~$30 | 1 config change |
| 2 | Reduce heartbeat to every 4hr + use Haiku | ~$40 | 1 cron edit |
| 3 | Reduce Radar to 3x/week + use Sonnet | ~$7 | 1 cron edit |
| 4 | Route subagents to Sonnet/Haiku | ~$10 | Config change |
| 5 | Compress cron prompts + concise output | ~$3 | Prompt editing |

**Combined: ~$70/mo savings → from ~$94/mo to ~$24/mo**

---

*This report should be revisited monthly as model pricing changes frequently. Anthropic and OpenAI typically reduce prices 1-2x per year.*
