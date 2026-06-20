# Watchdog Cron — Silent-when-OK Pattern

## Principle
For long-running services, cron jobs should check health silently and **only contact the user on failure**. Silence = everything is fine. Spam trains the user to ignore alerts.

## Cron Job Template

### Via Hermes cron
```bash
hermes cron create \
  --name "Claw_Trade Watchdog" \
  --schedule "every 5h" \
  --enabled-toolsets "terminal,file" \
  --prompt "
Check Claw_Trade live trading system and ONLY report if something is wrong:

1. Check container: docker inspect claw-trade-mt5 --format '{{.State.Status}}'
2. Check mt5linux: docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep -q 8001 && echo OK || echo DOWN'
3. Check live trading: cat /root/Claw_Trade/live_trading.pid | xargs -I{} kill -0 {} && echo OK || echo DOWN
4. If ANY check fails → run watchdog.py, read live_trading_output.log, fix it, report to user in Thai
5. If ALL pass → DO NOT SEND ANYTHING. Stay silent.
"
```

### Via system cron
```bash
# Every 5 hours: run watchdog, only stdout if there's a problem
0 */5 * * * cd /root/Claw_Trade && python3 watchdog.py 2>&1 | grep -v "already running" || true
```

## Three-tier Health Check
| Tier | What to check | Recovery |
|------|--------------|----------|
| 1. Docker container | `docker inspect` → Status=running | `docker compose up -d` |
| 2. mt5linux (inside container) | `ss -tlnp \| grep 8001` | Container restart (s6 auto-fixes) |
| 3. Live trading (host Python) | PID file + `kill -0` | `python3 watchdog.py` |