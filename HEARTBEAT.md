# HEARTBEAT.md — System Health Check

Run these checks periodically to ensure the environment is healthy.

## 1. Disk Usage
```bash
df -h / | tail -1
```
**Alert if:** usage exceeds 85%.

## 2. OpenClaw Gateway
```bash
openclaw gateway status
```
**Expect:** `Runtime: running` and `RPC probe: ok`.

## 3. Workspace Integrity
Check that required infrastructure files exist:
```bash
for f in audit.jsonl failsafe.jsonl memory/cost_ledger.jsonl memory/episodes.jsonl memory/friction.jsonl proactive/interventions.jsonl; do
  [ -f "/Users/macbook/.openclaw/workspace/$f" ] && echo "OK: $f" || echo "MISSING: $f"
done
[ -d "/Users/macbook/.openclaw/workspace/runs" ] && echo "OK: runs/" || echo "MISSING: runs/"
```

## 4. Pending Issues
- Check for stale files in `runs/undelivered/`
- Review latest `memory/*.md` for unresolved action items
- Verify cron jobs: `openclaw cron list`

## Current Snapshot (2026-02-13)
- **Disk:** 6% used (11Gi / 228Gi) ✅
- **Gateway:** Running, pid 20122, RPC ok ✅
- **Workspace files:** All present ✅
