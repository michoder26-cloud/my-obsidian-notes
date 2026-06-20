---
name: claw-trade-mt5-linux
description: Deploy and manage Claw_Trade XAU/USD trading bot on Linux using Docker + mt5linux (no Windows VPS)
---

# Claw_Trade MT5 on Linux (Docker + mt5linux)

## Overview
Run MetaTrader 5 on Linux via Docker (gmag11/metatrader5_vnc) with mt5linux RPyC bridge. No Windows VPS needed.

## Architecture
- **Docker container**: gmag11/metatrader5_vnc (Debian 12 + Wine + MT5 + KasmVNC)
- **mt5linux**: Python RPyC server inside container (port 8001) — bridge between host Python and MT5
- **Host Python**: Uses `from mt5linux import MetaTrader5` via RPyC to communicate with container MT5
- **VNC WebUI**: Port 3000 (KasmVNC nginx → Xvnc websocket :6901) for manual MT5 login
- **Xvnc (Virtual X server)**: Runs inside container, renders full Xfce desktop headlessly. The "Xorg" the user sees in the browser IS Xvnc — a virtual display server. It persists even when browser closes.

```
                    Host (AWS EC2)
┌──────────────────────────────────────────────────────────────┐
│  analyst (SSH terminal)                                      │
│  cd /root/Claw_Trade && python3 main.py live ...             │
│                                                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │ Docker Container                                 │        │
│  │                                                 │        │
│  │  Xvnc (:1) — virtual X server (headless)        │        │
│  │    └── Xfce Desktop (GUI โต๊ะทำงาน)              │        │
│  │          └── 🏦 MT5 Terminal (Wine)             │        │
│  │                                                 │        │
│  │  mt5linux RPyC server :8001 ◄── host Python bot │        │
│  │                                                 │        │
│  │  KasmVNC nginx :3000 ──► browser = desktop GUI  │        │
│  └─────────────────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────────┘
```

**Key insight for user Q&A:** The KasmVNC web desktop (port 3000) connects to the SAME Xvnc session running MT5 inside the container. The host user `analyst` is on a different system entirely — SSH terminal only, no GUI. The two connect ONLY through mt5linux (port 8001 RPyC).

## Setup Steps

### 1. Docker Compose
```yaml
version: '3'
services:
  mt5:
    image: gmag11/metatrader5_vnc
    container_name: claw-trade-mt5
    restart: unless-stopped
    ports:
      - "3000:3000"   # KasmVNC Web UI (browser remote desktop)
      - "8001:8001"   # RPyC API (mt5linux bridge)
    volumes:
      - mt5_config:/config
    environment:
      - CUSTOM_USER=trader
      - PASSWORD=change_me_pls
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 120s

volumes:
  mt5_config:
```

### 2. Start Container
```bash
docker compose up -d
```

### 3. Login to MT5 via VNC
- Open http://YOUR_IP:3000 in browser
- Login: trader / clawtrade2026
- Open MT5 → File → Login to Trade Account → Enter FBS credentials

### 4. Start mt5linux Server (inside container)
```bash
docker exec --user abc claw-trade-mt5 bash -c '
  export WINEPREFIX=/config/.wine
  wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -m mt5linux --host 0.0.0.0 --port 8001
'
```

### 5. Auto-start mt5linux (s6 service)
Create `/run/service/mt5linux/run` inside container:
```bash
#!/usr/bin/with-contenv bash
export WINEPREFIX=/config/.wine
exec su - abc -c "export WINEPREFIX=/config/.wine && wine \"C:\\\\Program Files (x86)\\\\Python39-32\\\\python.exe\" -m mt5linux --host 0.0.0.0 --port 8001"
```

**⚠️ CRITICAL: Must run as user `abc`, NOT root.** The Wine prefix `/config/.wine` is owned by `abc`. If s6 runs as `root`, wine silently fails with `wine: '/config/.wine' is not owned by you` and mt5linux never starts. Always use `su - abc -c "..."` wrapper.

**Verify it worked:**
```bash
docker exec claw-trade-mt5 ps aux | grep python.exe
# Should show: C:\Program Files (x86)\Python39-32\python.exe -m mt5linux --host 0.0.0.0 --port 8001
# If only s6-supervise shows but no python.exe → check ownership/permission
```

**🔴 CRITICAL: Wine Permission Denied — Silent Failure Pattern**

If `/config/.wine` is owned by user `abc` but s6 runs the service as `root`, wine exits with:
```
wine: '/config/.wine' is not owned by you
```
s6-supervise still shows the service as "up" but no python.exe process exists.

**Diagnosis:**
```bash
# Check ownership
docker exec claw-trade-mt5 ls -la /config/.wine
# If owned by abc:abc → must run mt5linux as abc

# Check if python.exe is actually running
docker exec claw-trade-mt5 ps aux | grep python.exe
# If empty → mt5linux silently failed
```

**Fix — Two approaches:**

1. **Modify s6 run script** (persistent across container restarts):
```bash
docker exec claw-trade-mt5 bash -c 'cat > /run/service/mt5linux/run << '"'"'EOF'"'"'
#!/usr/bin/with-contenv bash
export WINEPREFIX=/config/.wine
exec su - abc -c "export WINEPREFIX=/config/.wine && wine \"C:\\\\Program Files (x86)\\\\Python39-32\\\\python.exe\" -m mt5linux --host 0.0.0.0 --port 8001"
EOF
chmod +x /run/service/mt5linux/run'
```

2. **Manual start** (temporary, lost on container restart):
```bash
docker exec -d claw-trade-mt5 bash -c "su - abc -c 'export WINEPREFIX=/config/.wine && wine \"C:\\\\Program Files (x86)\\\\Python39-32\\\\python.exe\" -m mt5linux --host 0.0.0.0 --port 8001'"
```

**After fix, verify:**
```bash
sleep 10
docker exec claw-trade-mt5 ps aux | grep python.exe
# Should show python.exe running
```

Then restart s6 supervision:
```bash
docker exec claw-trade-mt5 s6-svc -d /run/service/mt5linux
sleep 2
docker exec claw-trade-mt5 s6-svc -u /run/service/mt5linux
```

### 6. Host Python Setup
```bash
pip install mt5linux rpyc
```

### 7. mt5_connector.py Pattern
```python
from mt5linux import MetaTrader5 as _MT5Client
mt5 = _MT5Client(host="127.0.0.1", port=8001)
```

## Critical Fixes

### Symbol Name
FBS uses `XAUUSDc` not `XAUUSD`. Set in .env: `MT5_SYMBOL=XAUUSDc`

### get_account_info() - mt5linux returns tuple
```python
if isinstance(info, (tuple, list)):
    return {
        'balance': info[8],
        'equity': info[11],
        'name': info[18],
        'server': info[16],
        ...
    }
```

### import MetaTrader5 in orchestrator (TWO spots)
The `_monitor_live_positions()` method in `src/orchestrator.py` has TWO places that directly reference `MetaTrader5` — both break on Linux.

**Spot 1 — import (line ~1006):**
```python
# BEFORE (breaks on Linux):
        import MetaTrader5 as mt5

# AFTER:
        from mt5_connector import mt5 as _mt5
```

**Spot 2 — initialize call (line ~1147):**
```python
# BEFORE (mt5linux doesn't need initialize for history):
            if mt5.initialize():
                deals = mt5.history_deals_get(position=ticket)

# AFTER:
            if True:  # Already connected via mt5_connector
                deals = _mt5.history_deals_get(position=ticket)
```

**Spot 3 — constant reference (line ~1161):**
```python
# BEFORE:
                    if deal.entry == mt5.DEAL_ENTRY_OUT or deal.entry == 1:

# AFTER:
                    if deal.entry == _mt5.DEAL_ENTRY_OUT or deal.entry == 1:
```

### numpy compatibility
Inside container (Wine Python 3.9): Use numpy 2.0.2 (not 1.x)
```bash
pip install numpy==2.0.2
```

## Watchdog + Cron (silent-when-OK pattern)

### Watchdog Script
Use `/root/Claw_Trade/watchdog.py` to auto-restart live trading if it dies.
```bash
python3 watchdog.py
```

### Cron Job (silent-when-OK — preferred)
The cron job should check container, mt5linux, and live-trading process, but ONLY report to the user if something is wrong. Silence = all good.

```yaml
schedule: "every 5h"   # or "every 5m" for aggressive monitoring
deliver: "origin"      # reports to the chat where cron was created
prompt: |
  Check container, mt5linux, and live trading process.
  If ANY check fails → run watchdog.py, fix it, report in Thai.
  If ALL checks pass → DO NOT SEND ANY MESSAGE. Stay silent.
```

**Why silent:** The user explicitly prefers no spam. Only real problems warrant a notification.

## Commands
```bash
# Backtest (mock AI, no MT5 needed)
USE_MOCK_AI=true python main.py backtest --start-date 2026-05-01 --end-date 2026-05-31 --interval 1h

# Paper trading (simulated on MT5 Demo)
USE_MOCK_AI=true python main.py paper --interval 60

# Live trading (real AI, real orders)
python main.py live --confirm --interval 60

# Watchdog
python watchdog.py
```

## ⚠️ CRITICAL SECURITY: KasmVNC Port 3000

The container's KasmVNC server (port 3000) launches with **NO AUTHENTICATION by default**:
```
Xvnc -disableBasicAuth -SecurityTypes None -AlwaysShared
```

This means **anyone who knows the URL can access the MT5 desktop** without any password — the web interface loads directly into the VNC session with no login screen.

**Mitigation:**
```bash
# BLOCK port 3000 on host firewall (doesn't affect bot trading)
iptables -A INPUT -p tcp --dport 3000 -j DROP

# Or better: use the docker-compose.yml to NOT expose port 3000 at all
# (only port 8001 is needed for trading)
```

The bot does NOT need port 3000 to function — it trades via mt5linux API (port 8001). Keep it blocked unless you're doing manual MT5 operations.

## 🔴 xrdp REMOVED — Use NoMachine Instead

**As of June 2026, xrdp was removed** from this setup. xrdp was too laggy on AWS (software render + latency). The user now uses **NoMachine** (port 4000) which is:

| Protocol | Speed | Auth | Status |
|:---------|:-----|:----:|:------:|
| ❌ **xrdp** (port 3389) | Laggy | Username+Password | ✅ Removed |
| ✅ **NoMachine** (port 4000) | Fast (60fps) | Username+Password (SSH-based) | ✅ Active |
| ✅ **SSH terminal** (port 22) | Instant | SSH key | ✅ Always available |

NoMachine uses NX protocol (better compression + smart caching). Install:
```bash
wget https://download.nomachine.com/download/8.16/Linux/nomachine_8.16.1_1_amd64.deb
sudo dpkg -i nomachine_8.16.1_1_amd64.deb
```
Then open NoMachine client → New Connection → host IP (port 4000) → user `analyst`.

### Legacy xrdp Info (for historical reference)

The host **used to** run xrdp (port 3389) so `analyst` could connect via Windows Remote Desktop Connection → full Xfce desktop. This was SEPARATE from KasmVNC in Docker.

### Two Desktop Systems — Don't Confuse!

| Feature | ~~xrdp (port 3389)~~ [REMOVED] | KasmVNC (port 3000) [BLOCKED] |
|:--------|:---------------------------:|:---------------------------:|
| **User** | `analyst` (host) | `abc` (in Docker) |
| **X Server** | Xorg on host | Xvnc on container display :1 |
| **Access method** | ~~Remote Desktop~~ → **Use NoMachine (port 4000)** | ~~Browser~~ → **Blocked on firewall** |
| **Can do** | Edit code, run bot, git, terminal | Open MT5 to login/manage |
| **Cannot do** | Open MT5 (not installed on host) | Run Python bot, git, terminal |
| **They connect via** | Nothing — completely separate | mt5linux (port 8001 RPyC) |

```
┌─────────────────────────────────────────────────────────────┐
│   HOST (AWS EC2)                        DOCKER CONTAINER    │
│                                        ┌──────────────────┐│
│  NoMachine :4000                       │ KasmVNC :3000    ││
│  Xorg :11  user: analyst               │ Xvnc :1 user: abc││
│  ┌───────────────────┐                 │ ┌──────────────┐ ││
│  │ Remote Desktop    │                 │ │ 🏦 MT5       │ ││
│  │ ≡ Terminal, code, │  mt5linux:8001  │ │ Terminal     │ ││
│  │   git, bot        │◄─── RPyC ──────┤ │ (XAUUSDc)   │ ││
│  └───────────────────┘                 │ └──────────────┘ ││
│                                        └──────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## Troubleshooting

See [references/mt5-docker-debug.md](references/mt5-docker-debug.md) for quick health checks, common error patterns, and recovery time estimates.

### 🔴 Bot Not Trading — "Symbol not found" Loop

**Symptoms:** Log shows repeated `Symbol 'XAUUSDc' not found` + `Could not find any valid Gold symbol` every 10 seconds. Zero trades opened.

**Root cause:** MT5 inside Docker is either:
1. Still starting up (mt5linux server not yet ready)
2. Stuck in a restart loop (`start.sh` spawning many times)
3. MT5 terminal not logged into broker account

**Diagnosis steps:**
```bash
# 1. Check if mt5linux is actually running inside container
docker exec claw-trade-mt5 ps aux | grep mt5linux
# Should show: python.exe -m mt5linux --host 0.0.0.0 --port 8001
# If only s6-supervise shows but no python.exe → mt5linux crashed

# 2. Check for restart loops
ps aux | grep "start.sh" | grep -v grep | wc -l
# If >5 → MT5 is stuck restarting

# 3. Test connection from host
cd /root/Claw_Trade && .venv/bin/python3 -c "
from mt5linux import MetaTrader5
m = MetaTrader5()
m.connect()
syms = m.symbols_get()
gold = [s for s in syms if 'GOLD' in s.name.upper() or 'XAU' in s.name.upper()]
print('Gold symbols:', [s.name for s in gold])
print('Total:', len(syms))
"
# ConnectionRefusedError → mt5linux not running
# Empty gold list → MT5 not logged into broker
```

**Fix path:**
1. If mt5linux not running → start it: `docker exec -d claw-trade-mt5 bash -c 'export WINEPREFIX=/config/.wine && wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -m mt5linux --host 0.0.0.0 --port 8001 &'`
2. If start.sh loop → `pkill -f start.sh` on host, then `docker restart claw-trade-mt5`
3. If symbols empty → open VNC (port 3000), login to MT5 broker account, add XAUUSDc to Market Watch

**Recovery time:** ~30-60 seconds after MT5 fully loads and symbols appear.

### 🟡 Dashboard Port 8080 Not Responding

**Symptoms:** `curl http://localhost:8080/` returns empty or connection refused.

**Fix:** Dashboard server is separate from trading bot. Restart with:
```bash
pkill -f dashboard_server.py; sleep 1
cd /root/Claw_Trade && python3 dashboard_server.py &
```

### 🟡 OpenRouter 401 — "Missing Authentication Header"

**Symptoms:** Bot logs `HTTP 401: Missing Authentication header` → CEO agent rejects all signals → NO_TRADE.

**Do NOT assume the API key expired.** The most common causes are:
1. `.env` file not loaded into bot process env vars
2. `load_dotenv()` not called or missing `override=True`

**Diagnosis:**
```bash
# Check if load_dotenv is in config
grep -n "load_dotenv" /root/Claw_Trade/src/config.py

# Test if key loads correctly
cd /root/Claw_Trade && .venv/bin/python3 -c "
from dotenv import load_dotenv
load_dotenv(override=True)
import os
key = os.getenv('OPENROUTER_API_KEY', 'MISSING')
print('Key present:', bool(key) and key != 'MISSING')
print('Key prefix:', key[:10] + '...' if key and key != 'MISSING' else 'N/A')
"
```

**Fix:**
1. Ensure `load_dotenv(override=True)` is at the TOP of `src/config.py` (before any `os.getenv` calls)
2. Kill old bot process: `pkill -9 -f "main.py live"`
3. Restart bot fresh
4. Verify no more 401 errors in log

## Pitfalls
- **Container restart → mt5linux auto-starts via s6**, but MT5 terminal may need VNC re-login if the volume was rebuilt
- **Weekend/low liquidity → bot blocks trades** (LOW_LIQUIDITY regime filter) — this is correct safety behaviour
- **Yahoo Finance data limited to 60 days for 1h interval** — fallback when MT5 history fails
- mt5linux `copy_rates_range` may have tz_convert issues → falls back to Yahoo Finance (non-critical)
- **mt5linux can fail silently after container restart** — s6-supervise shows as running but the actual python.exe process inside Wine dies. Always verify with `docker exec claw-trade-mt5 ps aux | grep python.exe`
- **🔴 Wine permission denied — mt5linux silently fails**: If `/config/.wine` is owned by user `abc` but s6 runs the service as `root`, wine exits with `'/config/.wine' is not owned by you`. s6-supervise still shows the service as "up" but no python.exe process exists. **Fix**: Ensure the s6 run script uses `su - abc -c "..."` to run wine as the correct user. Check with `docker exec claw-trade-mt5 ls -la /config/.wine` to confirm ownership.
- **🔴 Bot running 2 instances (duplicate PIDs)**: If bot was restarted without killing the old process, two instances run simultaneously → duplicate trades, competing MT5 connections. **Always kill before restart**: `pkill -9 -f "main.py live"` then verify with `ps aux | grep main.py | grep -v grep` shows only ONE process.
- **🔴 Wrong default symbol — bot can't find XAUUSD**: Default symbol in `mt5_connector.py` is `"XAUUSD"` but FBS demo uses `"XAUUSDc"`. Bot logs `Symbol 'XAUUSDc' not found` repeatedly and never trades. **Fix**: Change default in `src/mt5_connector.py` line 36 from `"XAUUSD"` to `"XAUUSDc"`, or set `MT5_SYMBOL=XAUUSDc` in `.env`.
- **Hermes terminal `&` backgrounding blocked**: Hermes rejects shell-level background wrappers (`nohup`, `disown`, trailing `&`). **Workaround**: Write a startup script (`/tmp/start_bot.sh`) with `exec python3 -u main.py live ...` and run via `terminal(background=true, command="bash /tmp/start_bot.sh")`.
- **🔴 `.env` not loaded → OpenRouter 401 (NOT key expiry)**: If bot logs 401, do NOT assume key expired. Test with Python one-liner first. Fix: ensure `load_dotenv(override=True)` in config.py, then fully restart bot (kill + start fresh).
- **🔴 VPS without GPU cannot run large open-weight models**: This EC2 has 0 GPUs, 12GB RAM, 6 CPU cores. Models like MiniMax-M3 (428B) need multi-GPU. Options: (1) API-based inference, (2) GPU instance upgrade, (3) smaller quantized models (≤8B).
- **Dashboard design — don't serve pixel art theme for entire site**: User wants ONLY Agent HQ to be pixel art. Dashboard and Trades tabs should use clean dark theme (Inter font, Chart.js, cards). Pixel art (Press Start 2P font, CRT scanlines, canvas sprites) is ONLY for Agent HQ page.
- **Dashboard CDN dependency**: Chart.js loaded from cdn.jsdelivr.net. If the user's router has no internet, charts won't render. Consider bundling Chart.js locally if this becomes a recurring issue.
- **Tool loop prevention:** When terminal commands fail 3+ times with different errors, STOP and summarize what you know. Do not keep retrying the same approach — switch to read_file or analyze existing logs instead
- **Before committing to GitHub:** run credential check on staged files:
  ```bash
  grep -n -i "password\|api_key\|token\|secret" $(git diff --cached --name-only) 2>/dev/null
  ```
  Never commit `.env`, `.env.bak`, or files containing real credentials.
- Docker compose VNC `PASSWORD` must be a placeholder like `change_me_pls` in the public repo — never the real password.

## GitHub Push Workflow (don't leak credentials!)

This user's repo is public. Before pushing any Linux-support changes:

1. **Update `.gitignore` first** — add `.env.bak`, `*.pid`, `*.log`, `live_trading*`
2. **Sanitize `docker-compose.yml`** — change VNC `PASSWORD` to placeholder like `change_me_pls`
3. **Update `.env.example`** — add new config keys (MT5_HOST, MT5_PORT, etc.), never include real values
4. **Stage only safe files:**
   ```bash
   git add src/mt5_connector.py src/orchestrator.py docker-compose.yml .env.example .gitignore LINUX_SETUP.md watchdog.py
   ```
5. **Verify no secrets in staged files:**
   ```bash
   grep -n -i "password\|api_key\|secret\|token" $(git diff --cached --name-only)
   ```
6. **Commit + Push** (if HTTPS + token auth):
   ```bash
   git commit -m "🐧 Add Linux support: Docker + mt5linux setup"
   # Set remote with token (one-time):
   git remote set-url origin https://USERNAME:TOKEN@github.com/USERNAME/REPO.git
   git push origin main
   # Reset remote URL to hide token:
   git remote set-url origin https://github.com/USERNAME/REPO.git
   ```

## User Preferences (applied to this skill)

This user:
- Prefers **silent monitoring** — cron reports ONLY on failure. Never spam.
- Wants **educational documentation** for GitHub audience — teach, don't just fix.
- Explains in **Thai with ASCII diagrams** when architecture questions arise.
- Needs **step-by-step hand-holding** for Git/GitHub — no prior credential setup.
- Values **reliability and memory** — hates being forgotten between sessions.