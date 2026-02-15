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
for f in memory/episodes.jsonl memory/friction.jsonl; do
  [ -f "$f" ] && echo "OK: $f" || echo "MISSING: $f"
done
[ -d "runs" ] && echo "OK: runs/" || echo "MISSING: runs/"
```

## 4. Pending Work
- Check for stale items in `runs/`
- Review latest `memory/*.md` for unresolved action items
- Verify worker agents are responsive
