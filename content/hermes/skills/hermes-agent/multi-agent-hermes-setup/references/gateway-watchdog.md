# Gateway Watchdog — Auto-Restart + Resilience

## Problem
Multi-agent gateways (trader, coder, news, system) can crash silently. User
only discovers when bots stop responding in Telegram. Need auto-restart that
**survives high load, watchdog death, and crash loops**.

## The Three-Layer Defense

```
┌─────────────────────────────────────────────────┐
│ Layer 1: systemd (hermes-gateway-watchdog.service) │
│   - Auto-restarts the watchdog itself if it dies │
│   - Resource limits (MemoryMax=200M, CPUQuota=20%)│
│   - Survives reboot (WantedBy=multi-user.target) │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│ Layer 2: watchdog.sh daemon                      │
│   - Checks all 4 gateway processes every 30s    │
│   - Restarts any crashed gateway                │
│   - SILENT — no Telegram spam (user hates it)  │
│   - Load-threshold + cooldown protection        │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│ Layer 3: crontab @reboot (backup)                │
│   - Catches the case where systemd is broken     │
│   - Last-resort restart after VPS reboot        │
└─────────────────────────────────────────────────┘
```

## Layer 1: Systemd Service (CRITICAL — watchdogs die)

> ⚠️ **Real incident 2026-06-18 21:00**: watchdog died (load avg 92.47),
> 4 gateways went down, stayed down for hours until user noticed. This
> service prevents recurrence.

```bash
# /etc/systemd/system/hermes-gateway-watchdog.service
sudo tee /etc/systemd/system/hermes-gateway-watchdog.service > /dev/null << 'EOF'
[Unit]
Description=Hermes Multi-Gateway Watchdog (auto-restart on crash)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/gateway-watchdog.sh
Restart=always
RestartSec=10
StandardOutput=append:/var/log/gateway-watchdog/watchdog-systemd.log
StandardError=append:/var/log/gateway-watchdog/watchdog-systemd.log

# Resource limits - กันไม่ให้ watchdog กิน CPU/RAM มากเกิน
MemoryMax=200M
CPUQuota=20%

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable hermes-gateway-watchdog.service
sudo systemctl start hermes-gateway-watchdog.service

# Verify
systemctl is-active hermes-gateway-watchdog.service
```

### Killing the old ad-hoc watchdog
If watchdog was already running via `terminal(background=true)`:
```bash
# Kill old watchdog
pkill -f "bash /usr/local/bin/gateway-watchdog.sh"
sleep 2
# Then start systemd version
sudo systemctl start hermes-gateway-watchdog.service
```

### Verify systemd will actually restart it
```bash
# Find watchdog PID, kill it, wait 15s, check if it's back
watchdog_pid=$(pgrep -f "bash /usr/local/bin/gateway-watchdog" | head -1)
sudo kill -9 "$watchdog_pid"
sleep 15
pgrep -f "bash /usr/local/bin/gateway-watchdog" | head -1  # should be a NEW PID

# Check systemd journal
sudo journalctl -u hermes-gateway-watchdog.service -n 10 --no-pager
```

## Layer 2: Resilient Watchdog Script

The hardened version of `gateway-watchdog.sh` includes:

### Resource limits at top of script
```bash
# ulimit at start of /usr/local/bin/gateway-watchdog.sh
ulimit -t 60        # CPU time limit (seconds)
ulimit -v 200000    # Virtual memory limit (200MB)
ulimit -f 1024      # File size limit (1MB)
```

### Config additions
```bash
PROFILES=("trader" "coder" "news" "system")
CHECK_INTERVAL=30
LOAD_THRESHOLD=8   # ถ้า load > 8 ข้ามรอบนี้ (กัน thrash)
COOLDOWN_FILE="/var/run/gateway-watchdog/restart-cooldown"
COOLDOWN_SECONDS=300  # ห้าม restart ตัวเดิมซ้ำภายใน 5 นาที
```

### Main loop with load + cooldown protection
```bash
while true; do
  # Load threshold: skip restart if VPS is overloaded
  current_load=$(cut -d' ' -f1 /proc/loadavg 2>/dev/null || echo "0")
  load_int=${current_load%.*}
  if [ "${load_int:-0}" -gt "$LOAD_THRESHOLD" ]; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ⚠️  Load $current_load > $LOAD_THRESHOLD — skipping restart cycle" >> "$LOG_DIR/watchdog.log"
    sleep "$CHECK_INTERVAL"
    continue
  fi

  for profile in "${PROFILES[@]}"; do
    pid=$(get_gateway_pid "$profile")

    if [ -z "$pid" ]; then
      # Cooldown: prevent crash loops
      last_restart=0
      [ -f "$COOLDOWN_FILE" ] && last_restart=$(cat "$COOLDOWN_FILE" 2>/dev/null || echo 0)
      now=$(date +%s)
      if [ $((now - last_restart)) -lt "$COOLDOWN_SECONDS" ] && [ "$last_restart" -gt 0 ]; then
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] ⏸️  $profile DOWN but in cooldown (last restart $((now - last_restart))s ago)" >> "$LOG_DIR/watchdog.log"
        continue
      fi

      echo "[$(date '+%Y-%m-%d %H:%M:%S')] ❌ $profile gateway DOWN — restarting..." >> "$LOG_DIR/watchdog.log"
      start_gateway "$profile"
      echo "$now" > "$COOLDOWN_FILE"
    fi
  done
  sleep "$CHECK_INTERVAL"
done
```

## Layer 3: Backup crontab @reboot

```bash
# Catches the case where systemd is broken
(crontab -l 2>/dev/null | grep -v "gateway-watchdog"; \
 echo "@reboot sleep 30 && /usr/local/bin/gateway-watchdog.sh >> /var/log/gateway-watchdog/watchdog.log 2>&1 &") \
 | crontab -
```

## Logs and Diagnostics

### Log locations
- `/var/log/gateway-watchdog/watchdog.log` — main watchdog log
- `/var/log/gateway-watchdog/watchdog-systemd.log` — systemd-level log
- `/var/log/gateway-watchdog/<profile>-gateway.log` — per-profile stdout
- `/var/log/gateway-watchdog/<profile>-errors.log` — per-profile stderr
- `journalctl -u hermes-gateway-watchdog.service` — systemd journal

### Health check commands
```bash
# Are all 4 gateways running?
for p in trader coder news system; do
  pid=$(pgrep -f "hermes.*profile $p.*gateway" | head -1)
  [ -n "$pid" ] && echo "✅ $p (PID: $pid)" || echo "❌ $p - DOWN"
done

# Is watchdog running?
systemctl is-active hermes-gateway-watchdog.service

# Watchdog log tail
tail -30 /var/log/gateway-watchdog/watchdog.log

# Per-profile error log
tail -20 /var/log/gateway-watchdog/news-errors.log
```

## Pitfalls

- **WATCHDOG CAN DIE** — This is the most important lesson. A bare `terminal(background=true)` watchdog has no supervisor. If the watchdog itself crashes, NOTHING will restart the gateways. **Always wrap watchdog in systemd.**
- **Load > 8 = don't restart** — During high load, the system is fighting for resources. Spawning new gateway processes just makes it worse. Skip the restart cycle and let the system recover.
- **Crash loop = cooldown** — If a gateway crashes repeatedly (e.g. bad config), the watchdog will thrash. 5-minute cooldown prevents this.
- **Do NOT use `nohup`** — Hermes terminal blocks it. Use systemd or `terminal(background=true)`.
- **Watchdog must be SILENT** — User hates Telegram notification spam. Don't add Telegram alerts to the watchdog. Just restart quietly.
- **Systemd service needs network-online.target** — Gateways connect to Telegram/Ollama. If network not ready at boot, gateways fail to start. The `After=` directive handles this.

## Diagnostic: Diagnosing "gateways all down"

```bash
# Step 1: Is watchdog running?
systemctl is-active hermes-gateway-watchdog.service
# If 'inactive' → start it: sudo systemctl start hermes-gateway-watchdog.service

# Step 2: Why did watchdog die?
sudo journalctl -u hermes-gateway-watchdog.service -n 50 --no-pager
# Look for: signal=KILL, exit code, OOM, etc.

# Step 3: Is it a load problem?
uptime  # check load average
# If load > 8, see what's eating CPU: top -bn1 | head -20

# Step 4: Check gateway errors (last crashed gateway)
tail -50 /var/log/gateway-watchdog/<profile>-errors.log

# Step 5: Common crash causes
# - API key expired → 401 errors → check .env
# - Ollama rate limit → 429 errors → wait
# - Bad config → check ~/.hermes/profiles/<profile>/config.yaml
# - Disk full → df -h

# Step 6: Force restart all gateways
pkill -f "hermes.*gateway run"  # kill all
sleep 5
sudo systemctl restart hermes-gateway-watchdog.service  # watchdog will respawn
```
