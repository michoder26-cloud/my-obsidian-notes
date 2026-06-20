# Cron-Based Gateway Monitor (RECOMMENDED)

## Why cron, not systemd loops?

A systemd service running a `while true; do ... sleep 10; done` loop **WILL FAIL**:
- systemd tracks restart count — if the script restarts (or appears dead), systemd kills it after N restarts
- Real incident 2026-06-19: `hermes-auto-restart.service` restarted 7 times in minutes, then systemd gave up
- Bash `local` keyword errors outside functions crash the script silently
- Background processes spawned by the loop get killed when systemd restarts the service

**Cron is rock-solid**: cron daemon never dies, runs your script every N minutes, no persistent process to manage.

## Setup

### 1. Create the monitor script
```bash
#!/bin/bash
# /usr/local/bin/gateway-monitor.sh
export PATH="$HOME/.local/bin:/usr/local/lib/hermes-agent/venv/bin:$PATH"
LOG="/var/log/gateway-watchdog/cron-monitor.log"
PROFILES="trader coder news system"

for profile in $PROFILES; do
  pid=$(pgrep -f "hermes.*profile ${profile}.*gateway run" | head -1)
  if [ -z "$pid" ]; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ⚡ $profile DOWN — restarting..." >> "$LOG"
    rm -f $HOME/.hermes/profiles/$profile/gateway.lock
    rm -f $HOME/.hermes/profiles/$profile/gateway.pid
    pgrep -f "hermes.*profile ${profile}.*gateway" | xargs -r kill -9 2>/dev/null
    /usr/local/lib/hermes-agent/venv/bin/hermes --profile "$profile" gateway run \
      >> /var/log/gateway-watchdog/${profile}-gateway.log 2>&1 &
    sleep 8
    new_pid=$(pgrep -f "hermes.*profile ${profile}.*gateway" | head -1)
    if [ -n "$new_pid" ]; then
      echo "[$(date '+%Y-%m-%d %H:%M:%S')] ✅ $profile started (PID $new_pid)" >> "$LOG"
    else
      echo "[$(date '+%Y-%m-%d %H:%M:%S')] ❌ $profile failed" >> "$LOG"
    fi
  fi
done
```

### 2. Add to crontab (every 2 minutes)
```bash
chmod +x /usr/local/bin/gateway-monitor.sh
(crontab -l 2>/dev/null | grep -v "gateway-monitor"; \
 echo "*/2 * * * * /usr/local/bin/gateway-monitor.sh") | crontab -
```

### 3. Default gateway uses systemd (separate)
The `default` profile runs via systemd user service (`~/.config/systemd/user/hermes-gateway.service`)
with `Restart=always RestartSec=1`. This works because default is a single process, not a loop.

## Three-Layer Defense (final architecture)

| Layer | What | Covers | Status |
|-------|------|--------|--------|
| 1. systemd user service | `default` gateway only | Restart=always, RestartSec=1, OOMScoreAdjust=-500 | ✅ Works |
| 2. Cron `*/2` monitor | trader/coder/news/system | Checks every 2 min, restarts if down | ✅ Works |
| 3. Cron @reboot backup | All gateways | Starts everything after VPS reboot | ✅ Backup |

## What FAILED (do NOT use)
- ❌ `hermes-auto-restart.service` (systemd loop) — restart counter exceeded, gave up after 7 tries
- ❌ `gateway-watchdog.sh` with 5-min cooldown — cooldown too long, gateways stayed down
- ❌ `nohup ... &` inside Hermes terminal — blocked by terminal tool
- ❌ Shell `&` backgrounding in foreground terminal — blocked by terminal tool

## Pitfalls
- **`local` outside function**: Bash errors silently. Don't use `local` in the main loop body.
- **Stale lock/pid files**: Always `rm -f gateway.lock gateway.pid` before restart.
- **Hermes blocks backgrounding**: Use cron context (cron runs scripts detached) or `terminal(background=true)`.
- **Sleep too long in loop**: Each profile check should be quick. 8s sleep after start is enough.
