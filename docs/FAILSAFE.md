# FAILSAFE & ERROR RECOVERY v1.0

## GOVERNING PRINCIPLE

An autonomous agent that can't handle transient failures isn't autonomous — it's a fragile script. Every trigger must have a defined failure path.

## DEGRADATION TIERS

### Tier 1: Channel Failover

If the primary notification channel (Telegram) fails:
1. Retry once after 30 seconds
2. If still failing: log to `memory/failed_notifications.jsonl` with full payload
3. Attempt email fallback (if configured) with subject: `[KAI FAILOVER] <trigger_id>`
4. If all channels fail: write to local `runs/undelivered/` directory. Heartbeat checks this directory every hour and retries.

### Tier 2: API Failover

If Claude API returns an error:

| Error | Action |
|-------|--------|
| 429 (Rate Limit) | Queue the action. Retry with exponential backoff: 30s, 60s, 120s. Max 3 retries. |
| 500/502/503 (Server Error) | Retry once after 60s. If still failing, log and skip. |
| 401 (Auth Error) | Halt all LLM calls. Alert Shuki: "API key may be expired." |
| Timeout (>30s) | Retry once. If second attempt times out, log and skip. |
| Context too large | Trim context: remove episodic lessons first, then belief details, keeping only snapshot summaries. Retry with reduced context. |

### Tier 3: Data Source Failover

If an external data source is unavailable during a trigger:

| Source | Degraded Behavior |
|--------|-------------------|
| RepairDesk API down | Morning brief skips repair queue section, notes "RepairDesk unavailable" |
| GitHub API down | Morning brief skips issues section, notes "GitHub unavailable" |
| Calendar API down | Morning brief skips calendar section, notes "Calendar unavailable" |
| HubSpot webhook fails | Lead is logged to `runs/undelivered/` for manual processing |

**Rule**: Never fail the entire trigger because one data source is down. Complete what you can, note what's missing, deliver the partial result.

### Tier 4: Data Corruption Recovery

| Corrupted File | Recovery |
|---------------|----------|
| `memory/beliefs.snapshot.json` (fails to parse) | Rebuild from `memory/beliefs.jsonl` source of truth. Alert Shuki. |
| `schemas/governance.json` (hash mismatch) | Hard stop. All activity halts. Alert Shuki: "Governance file integrity violation." |
| `proactive/triggers.yaml` (parse error) | Fall back to last known good copy in `runs/config_backups/`. Alert Shuki. |
| `metrics/assertiveness.json` (fails to parse) | Reset to defaults: level=3, all metrics=0. Alert Shuki. |
| `memory/cost_ledger.jsonl` (corrupted) | Assume max spend for current period (conservative). Halt proactive actions. Alert Shuki. |

## PARTIAL EXECUTION HANDLING

If a multi-step trigger completes some steps but fails mid-way:

1. The checkpoint system (`runs/<run_id>/state.json`) records the last completed step
2. On next heartbeat, check for `status: "running"` checkpoints older than `stale_after_hours` (default: 2)
3. For stale checkpoints:
   - If `recovery_policy.on_stale == "resume"`: replay from last completed step
   - If `recovery_policy.on_stale == "summarize_and_notify"` (default): send Shuki a summary of what completed and what didn't, via Telegram. If Telegram is down, use Tier 1 channel failover.
   - If `recovery_policy.on_stale == "abort"`: mark checkpoint as failed, log, move on

## CASCADING TRIGGER PROTECTION

If multiple triggers fire simultaneously (e.g., 10 leads arrive at once from a marketing blast):

1. Triggers are queued in arrival order
2. Each trigger checks budget before execution
3. If hourly budget would be exceeded: queue remaining triggers for next hour window
4. **Critical**: Queued triggers that exceed their SLA deadline are escalated as "SLA breach — queued, not dropped" with count of pending items
5. Maximum queue depth: 20 triggers. Beyond 20: log overflow event, alert Shuki with count

## RE-ESCALATION

If Shuki doesn't respond to a Telegram ping within 20 minutes:
1. Send a second ping with escalation marker: "2nd notice"
2. If no response within 40 minutes total: log as `user_response: "ignored"`, mark lead as needing manual follow-up
3. Do NOT send more than 2 pings per trigger. Beyond that, it becomes counterproductive.

## CIRCUIT BREAKER RECOVERY

When `max_failed_actions_before_pause` (3) is hit:
1. All proactive actions pause
2. Heartbeat continues running (for kill switch checks and monitoring)
3. Alert Shuki: "Kai paused after 3 consecutive failures. Likely cause: [most recent error]. Reply RESUME to restart."
4. On "RESUME" from Shuki: reset failure counter, resume normal operation
5. Auto-resume after 2 hours if no response (with reduced trigger rate: 50% throttle for 1 hour)

## MONITORING & FAILSAFE LOG

Failsafe events are logged to `.openclaw/workspace/failsafe.jsonl`:
```json
{
  "ts": "2026-02-13T14:30:00Z",
  "event": "api_failure|channel_fallback|data_recovery|cascade_queue|circuit_breaker",
  "service": "claude_api|telegram|repairdesk|github|n8n",
  "details": "429 rate limit on lead draft",
  "recovery_action": "queued_retry_15s",
  "resolved": true
}
```

Weekly health summary includes failsafe event count. If > 5 failsafe events/week → flag in strategic drift review as potential infrastructure issue.
