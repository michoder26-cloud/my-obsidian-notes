---
name: hermes-system-ops
description: "Monitor and maintain multi-profile Hermes deployments: gateways, dashboards, trading bots, watchdogs, and auto-remediation."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, system-ops, monitoring, watchdog, multi-profile, devops, trading, mt5]
    related_skills: [hermes-agent]
---

# Hermes System Operations

Monitor and maintain a running multi-profile Hermes deployment. This skill covers:
- Health checks for gateways, dashboards, and bot processes
- Watchdog patterns (cron-based AI watchdog, shell daemons, reboot scripts)
- Auto-remediation: fix errors immediately, report on a schedule
- MT5 / ClawTrade specific debugging (mt5linux + Docker + Wine)
- Thai language support for system reports

## When to Use

- User asks you to be "System Agent" or "DevOps" for their Hermes setup
- User wants periodic health checks and auto-fix without being notified each time
- User has multiple Hermes profiles (trader, coder, news, system) running as gateways
- User has a trading bot (ClawTrade) that needs monitoring
- User says "check if everything is running" or "fix errors automatically"
- User prefers reports in Thai language

## Architecture Overview

A typical multi-profile deployment looks like:

```
Host machine
├── Hermes Profiles (4x)
│   ├── trader/   → gateway + Telegram bot + cron watchdog
│   ├── coder/    → gateway + Telegram bot
│   ├── news/     → gateway + Telegram bot
│   └── system/   → gateway + Telegram bot (this agent)
│
├── Hermes Dashboard (port 9119)
│
├── ClawTrade (trading bot)
│   ├── Docker container (claw-trade-mt5)
│   │   ├── Wine + MT5 terminal64.exe
│   │   └── RPyC server (port 8001, mt5linux)
│   ├── Python bot (main.py live)
│   ├── Watchdog (watchdog_v2.py, cron every 15min)
│   └── ClawTrade Dashboard (dashboard_server.py, port 8080)
│
├── gateway-watchdog.sh (shell daemon, every 30s)
└── hermes-services.sh (@reboot crontab)
```

## Health Check Procedure

### 1. Gateways

```bash
# Check all profiles
for p in trader coder news system; do
  state=$(python3 -c "import json; d=json.load(open(f'/root/.hermes/profiles/$p/gateway_state.json')); print(d['gateway_state'], d['platforms']['telegram']['state'])" 2>/dev/null)
  echo "$p: $state"
done
```

**Fix if down:**
```bash
hermes --profile <name> gateway run &
```

Or let `gateway-watchdog.sh` handle it (runs every 30s).

### 2. Dashboards & Auxiliary Services

**Hermes Dashboard (Port 9119):**
```bash
ss -tlnp | grep 9119
```

**ClawTrade Dashboard (Port 8080):**
```bash
ss -tlnp | grep 8080
```

**Pixel Agent Office (Port 9120):**
```bash
curl -s --max-time 3 http://localhost:9120/api/status | head -c 200
```
This is a Python HTTP server (`/root/pixel-agent-office/server.py`) with a p5.js
pixel-art frontend. It is **not** in `hermes-services.sh` and does NOT auto-start
after reboot — it must be started manually or added to the boot script.

**Fix if down:**
```bash
# Hermes
hermes dashboard --host 0.0.0.0 --port 9119 --no-open &

# ClawTrade
cd /root/Claw_Trade && python3 dashboard_server.py &

# Pixel Agent Office — must use terminal(background=true), not nohup/& in foreground
# In Hermes terminal tool:
#   terminal(background=true, command="cd /root/pixel-agent-office && python3 server.py &")
# Then verify after 2-3 seconds:
#   curl -s --max-time 3 http://localhost:9120/api/status
```

### 3. ClawTrade Bot

```bash
# Container status
docker ps --format "{{.Names}} {{.Status}}" | grep claw

# Bot process
ps aux | grep 'main.py live' | grep -v grep

# Latest log
ls -t /tmp/claw_live*.log | head -1 | xargs tail -30

# Watchdog log
tail -10 /root/Claw_Trade/watchdog_v2.log

# Errors
ls -t /tmp/claw_live*.log | head -1 | xargs grep 'ERROR' | tail -10
```

### 4. MT5 Connection (via Docker + Wine + RPyC)

```bash
# Check MT5 terminal in container
docker exec claw-trade-mt5 bash -c "ps aux | grep terminal64 | grep -v grep"

# Check RPyC server
docker exec claw-trade-mt5 bash -c "ss -tlnp | grep 8001"

# Test connection (with credentials)
python3 -c "
import rpyc
conn = rpyc.classic.connect('127.0.0.1', 8001)
mt5 = conn.modules['MetaTrader5']
mt5.initialize(login=106123714, password='...', server='FBSTradestone-Demo')
print(mt5.account_info())
mt5.shutdown()
conn.close()
"
```

## Auto-Remediation Patterns

### MT5 Terminal Not Running
```bash
docker exec -d -u abc -e DISPLAY=:1 -e WINEPREFIX=/config/.wine claw-trade-mt5 bash -c \
  'WINEDEBUG=-all wine "/config/.wine/drive_c/Program Files/MetaTrader 5/terminal64.exe" > /tmp/mt5_cron.log 2>&1'
# Wait 20-30s for terminal to auto-login with broker
```

### RPyC Server Not Listening
```bash
docker exec -d -u abc -e WINEPREFIX=/config/.wine claw-trade-mt5 bash -c \
  'wine "C:\Program Files (x86)\Python39-32\python.exe" -m mt5linux --host 0.0.0.0 --port 8001 > /tmp/rpyc.log 2>&1'
# Wait 10-15s
```

### Bot Process Dead
```bash
cd /root/Claw_Trade && python3 main.py live --confirm --symbol XAUUSDc --interval 5 > /tmp/claw_live7.log 2>&1 &
```

### Container Down
```bash
docker restart claw-trade-mt5
# Wait 30-60s for Wine + MT5 + RPyC to come up
```

### Gateway Down
```bash
hermes --profile <name> gateway run &
```

## Watchdog Hierarchy

| Watchdog | Type | Interval | What It Checks |
|----------|------|----------|----------------|
| ClawTrade AI Watchdog | Hermes cron (trader profile) | 15 min | Container, MT5 terminal, RPyC, bot, login |
| gateway-watchdog.sh | Shell daemon | 30s | All 4 gateway processes (trader, coder, news, system) |
| hermes-services.sh | @reboot crontab | Boot | Gateway + Dashboard start |
| System Agent Report | Hermes cron (system profile) | 5h | Full system report delivered to Telegram in Thai |

## Setting Up Periodic Reports

Use the `cronjob` tool to create a reporting schedule:

```
cronjob(action='create', schedule='every 5h', name='System Agent Report', prompt='...')
```

The prompt should be self-contained with exact bash commands to run for each check.
**Crucial:** For this specific user, the report output should be in **Thai**.

Delivery goes to the Telegram chat where the user asked for monitoring.

## Pitfalls

### mt5linux: initialize() vs login()

**WRONG** (causes "Terminal: Authorization failed" or "No IPC connection"):
```python
mt5.initialize()          # No credentials
mt5.login(login, password, server)  # Separate call — fails on mt5linux
```

**CORRECT** (works with mt5linux/Wine/Docker):
```python
mt5.initialize(login=login_num, password=password, server=server)  # All in one
```

mt5linux proxies calls through RPyC to a Wine-hosted MT5 terminal. The terminal
needs credentials at `initialize()` time, not via a separate `login()` call.
This is different from native MetaTrader5 on Windows.

### pandas 3.0 tz_convert() breaking change

mt5linux's `copy_rates_range()` internally calls `date.astimezone()`,
which breaks on pandas 3.0+ with tz-naive datetimes:
```
Error: tz_convert() takes exactly 2 positional arguments (1 given)
```

**Fix:** Convert datetimes to UTC-aware before passing to `copy_rates_range`:
```python
from datetime import timezone as dt_tz
start_dt = pd.to_datetime(start_date).to_pydatetime().replace(tzinfo=dt_tz.utc)
end_dt = pd.to_datetime(end_date).to_pydatetime().replace(tzinfo=dt_tz.utc)
rates = mt5.copy_rates_range(symbol, timeframe, start_dt, end_dt)
```

### gateway-watchdog.sh wrong profile name

The watchdog script lists profiles to monitor. If a profile name doesn't match
an actual `~/.hermes/profiles/<name>/` directory (e.g. using 'daemon' instead of
'system'), it will fail to start that gateway every 30 seconds, filling logs with
errors. Always verify the `PROFILES=()` array matches actual profile directories.

### MT5 terminal startup delay

After starting `terminal64.exe` in Wine or restarting the Docker container,
MT5 needs 20-60 seconds to:
1. Wine initializes
2. MT5 terminal loads
3. Auto-login with broker
4. RPyC server becomes ready

Don't try to connect immediately. Wait at least 20-30 seconds, then test.
The watchdog handles this with `time.sleep(20)` after each start command.

### Bot log file rotation

The bot writes to `/tmp/claw_live<N>.log`. Each restart increments N.
Always check the latest log file, not the old one. Use:
```bash
ls -t /tmp/claw_live*.log | head -1
```

### `hermes dashboard --status` false negative

`hermes dashboard --status` may report "No processes running" even when the
dashboard IS running and serving on port 9119. This is a known issue where the
status command doesn't detect processes started in certain modes.

**Reliable check:** Use `ss -tlnp | grep 9119` instead of `hermes dashboard --status`.
If port 9119 is LISTEN, the dashboard is running.

```bash
# Unreliable:
hermes dashboard --status   # May say "No processes running" falsely

# Reliable:
ss -tlnp | grep 9119        # Shows actual listening process
```

If the dashboard needs starting, it may say "already running on port 9119"
when you run `hermes dashboard` — trust the port check, not the status command.

### Hermes Dashboard build delay (vite build)

When starting `hermes dashboard`, it first builds the web UI with Vite (`tsc -b && vite build`).
This takes 20-30 seconds on first run. During this period:
- The process IS alive (visible in `ps aux`)
- Port 9119 is NOT yet listening
- `curl` returns HTTP 000 (connection refused)

This looks like a failure but is normal. The log shows:
```
→ Building web UI...
> tsc -b && vite build
...
✓ built in 2.45s
✓ Web UI built
HERMES_DASHBOARD_READY port=9119
```

**Fix:** Wait for `HERMES_DASHBOARD_READY` in the log, or poll `ss -tlnp | grep 9119` until it appears.
Do NOT kill and restart the process during the build — it will just start building again.

### Pausing / stopping an individual profile gateway

When a user asks to "stop" or "pause" a specific profile (e.g. "News ยังทำงานอยู่ให้เขาพัก"),
you cannot simply `kill` the gateway process — `gateway-watchdog.sh` will restart it
within 30 seconds. You must teach the watchdog to skip the profile first.

**Steps:**

1. **Patch `gateway-watchdog.sh`** to check for a `.paused` flag file per profile.
   Insert this block right after `pid=$(get_gateway_pid "$profile")` in the main loop:

   ```bash
   # Skip if profile is paused
   if [ -f "$PID_DIR/${profile}.paused" ]; then
     [ -n "$pid" ] && kill "$pid" 2>/dev/null
     continue
   fi
   ```

2. **Create the pause flag:**
   ```bash
   touch /var/run/gateway-watchdog/news.paused
   ```

3. **Kill the gateway process:**
   ```bash
   pkill -f "hermes.*--profile <name>.*gateway run"
   ```

4. **Restart the watchdog service** so it loads the patched script:
   ```bash
   systemctl restart hermes-gateway-watchdog.service
   ```
   This is critical — the old in-memory script won't have the pause check yet.
   Without the restart, the watchdog keeps restarting the killed gateway.

5. **Verify** after ~10 seconds that the profile stays down:
   ```bash
   ps aux | grep "hermes.*--profile <name>" | grep -v grep
   ```
   Also check the watchdog log to confirm it's skipping, not restarting:
   ```bash
   tail -5 /var/log/gateway-watchdog/watchdog.log
   ```

**Pitfall:** If you kill the gateway before patching+restarting the watchdog,
it will restart the gateway in a race — you may need to kill it 2-3 times
before the watchdog picks up the new script. The correct order is:
patch script → create flag → restart watchdog → then kill the gateway.

**To resume a paused profile:**
```bash
rm /var/run/gateway-watchdog/<name>.paused
```
The watchdog will detect the missing flag on its next 30s cycle and start
the gateway automatically.

### Auto-detect new agents without being told

When the user says "if new agents are added, check them too — I shouldn't have
to tell you", always scan for new Hermes profiles dynamically rather than
relying on a hardcoded list:

```bash
# Dynamic: find ALL profiles
ls /root/.hermes/profiles/
# Static (may miss new ones):
for p in trader coder news system; do ...; done
```

Build the profile list at runtime from the filesystem so new agents are
automatically included in health checks.

### Terminal background command restrictions

Hermes terminal tool blocks `nohup`, `disown`, `setsid`, and trailing `&` in
foreground mode. To start long-running processes:
- Use `terminal(background=true)` for the bot process
- Or let the watchdog/cron handle it (preferred — more resilient)

## Reporting Format

When delivering periodic reports, use this structure:

```
📊 **System Report** [date time]

**Gateways:**
- Trader: ✅/❌ [status]
- Coder: ✅/❌
- News: ✅/❌
- System: ✅/❌

**Dashboards:**
- Hermes (9119): ✅/❌
- ClawTrade (8080): ✅/❌

**ClawTrade:**
- Container: ✅/❌
- Bot: ✅/❌ [PID]
- MT5 Login: ✅/❌
- Trades Today: [count]
- Balance: [amount]
- Errors: [list or "none"]

**Fixed this period:** [list or "none"]
```

**Note:** For Thai users, translate the headers and status descriptions to Thai.

## References

See `references/mt5linux-debugging.md` for detailed MT5/Docker/Wine debugging recipes
and exact error messages with fixes.

See `templates/status-api-server.py` for a ready-to-run Python HTTP server that serves
a visual monitoring page alongside a `/api/status` JSON endpoint — checks all gateways,
Docker containers, dashboard, watchdog, and system resources in real time. Use this when
the user wants a custom visual dashboard (pixel art, retro UI, etc.) overlaid on live
backend status.

See `references/visual-monitoring-dashboard.md` for the full pattern: combining a Python
status API server with a p5.js visual frontend, including how to delegate the visual
frontend to a Coder subagent, the JSON status shape, and pitfalls specific to this
workflow (server startup, CORS, canvas screenshot limitations).

See `references/vps-to-pi5-migration-analysis.md` for measured RAM usage of all
deployment components, Raspberry Pi 5 vs VPS comparison, MT5/ARM constraints,
and hardware recommendations if the user considers migrating from VPS to Pi 5
or Mini PC.