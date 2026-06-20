---
name: clawtrade-mt5
description: |
  Manage, troubleshoot, and monitor the ClawTrade XAU/USD multi-agent MT5 trading bot
  running on Linux via Docker + Wine + mt5linux RPyC. Covers startup, recovery, watchdog,
  and code fixes for the mt5linux integration layer.
tags:
  - trading
  - mt5
  - xauusd
  - docker
  - wine
  - rpyc
  - watchdog
---

# ClawTrade MT5 Bot Management

## System Architecture

```
Host (Linux)
├── main.py live --confirm --symbol XAUUSDc --interval 5
│   └── src/orchestrator.py (6 AI agents via OpenRouter)
│       └── src/mt5_connector.py
│           └── mt5linux (RPyC client) ── port 8001 ──┐
└── Docker: claw-trade-mt5 (gmag11/metatrader5_vnc)   │
    ├── Wine: terminal64.exe (MT5 terminal)          │
    ├── Python39-32: mt5linux RPyC server ◄──────────┘
    ├── nginx (VNC web UI on port 3000)
    └── s6 services (auto-restart supervisors)
```

## Key Paths
- Project: `/root/Claw_Trade/`
- Bot entry: `main.py live --confirm --symbol XAUUSDc --interval 5`
- Config: `.env` (MT5_LOGIN, MT5_PASSWORD, MT5_SERVER, OPENROUTER_API_KEY)
- Trade DB: `trade_memory.db` (SQLite — trades, P&L, regime)
- Learned config: `learned_config.json` (self-optimized thresholds)
- Positions: `live_positions.json`
- Bot log: `/tmp/claw_live5.log`
- Watchdog: `watchdog_v2.py` (also at `~/.hermes/profiles/trader/scripts/clawtrade_watchdog.py`)

## CRITICAL: Don't break a working system

The user said "จำไว้นะครับผมไม่อยากมานั่งทำใหม่" ("remember, I don't want to sit here redoing this"). This session, a working MT5 bot was destroyed by aggressive Wine process killing during a routine health check, forcing a full container+volume rebuild + manual MT5 re-login via VNC.

**Rules:**
1. **Never `kill -9` Wine system processes** (wineserver, wineboot, winedevice) — this permanently corrupts the Wine prefix
2. If something is stuck, **`docker restart claw-trade-mt5`** the whole container, don't kill individual processes
3. Before touching Wine/MT5, take a screenshot first to see what state it's in
4. If you don't know what a process does, don't kill it — restart the container instead
5. The watchdog cron handles auto-recovery; manual intervention should be last resort

## Startup Sequence (5 layers, bottom-up)

1. **Docker container** — `docker start claw-trade-mt5` (or `docker compose up -d`)
2. **Kill xmrig** — the `gmag11/metatrader5_vnc` image has a crypto miner; always kill it:
   `docker exec claw-trade-mt5 pkill -9 -f xmrig; docker exec claw-trade-mt5 pkill -9 -f xmr_linux`
3. **MT5 terminal in Wine** — does NOT auto-start; must launch manually:
   ```
   docker exec -d -u abc -e DISPLAY=:1 -e WINEPREFIX=/config/.wine \
     claw-trade-mt5 bash -c 'WINEDEBUG=-all wine "/config/.wine/drive_c/Program Files/MetaTrader 5/terminal64.exe" > /tmp/mt5.log 2>&1'
   ```
   Wait ~15-20s for MT5 to initialize.
4. **mt5linux RPyC server** (port 8001) — s6 service usually starts it, but may fail if MT5 wasn't ready. Kill any stale instances first, then start fresh:
   ```
   docker exec claw-trade-mt5 bash -c 'kill $(pgrep -f "server.py") 2>/dev/null; kill $(pgrep -f "mt5linux") 2>/dev/null'
   sleep 2
   docker exec -d -u abc -e WINEPREFIX=/config/.wine -e WINEDEBUG=-all \
     claw-trade-mt5 bash -c 'python3 -m mt5linux --host 0.0.0.0 --port 8001 -w wine "C:\Program Files (x86)\Python39-32\python.exe" > /tmp/mt5linux.log 2>&1'
   ```
   Wait ~15s, then verify: `docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep 8001'`
5. **Bot process** — `cd /root/Claw_Trade && python3 main.py live --confirm --symbol XAUUSDc --interval 5`

## Critical Fixes (already applied — keep if re-cloning)

### Fix 1: numpy import in mt5linux RPyC namespace
**File:** `mt5linux/metatrader5.py` (site-packages)
**Problem:** `mt5.order_send()` triggers RPyC `eval()` on the Wine side, which references `np` but the RPyC namespace doesn't import numpy → `NameError: name 'np' is not defined`.
**Fix:** Add `self.__conn.execute("import numpy as np")` after the existing `import datetime` line in `MetaTrader5.__init__()`.

### Fix 2: dict vs attribute access for account_info
**File:** `src/orchestrator.py` (~line 676)
**Problem:** `MT5Connector.get_account_info()` always returns a `dict`, but orchestrator calls `acc_info.balance` (attribute) → `'dict' object has no attribute 'balance'`.
**Fix:** Use `acc_info['balance'] if isinstance(acc_info, dict) else getattr(acc_info, 'balance', 10000.0)`.

### Fix 3: OpenRouter API key must be real
**File:** `.env`
**Problem:** `OPENROUTER_API_KEY=***` (placeholder) → all 6 agents get HTTP 401.
**Fix:** Replace with a real OpenRouter API key (free tier works).

## Agents (6 — all use OpenRouter LLM)

| Agent | Role | Model |
|-------|------|-------|
| 🔮 QUANT | Technical Analyst (RSI/MACD/EMA) | meta-llama/llama-3-70b-instruct |
| 📡 NEWS | News Scout (Economic Calendar) | same |
| ⚔️ BULL | Bullish Strategist (BUY args) | same |
| 🛡️ BEAR | Bearish Guardian (SELL args) | same |
| 👑 CEO | Executive Chairman (Final Decision) | same |
| 🧬 LEARN | Learning Engine (Self-Optimization) | same |

CEO decision threshold: confidence ≥ 0.78 (TRENDING) / 0.82 (RANGING) / 0.85 (HIGH_VOL) / 0.90 (LOW_LIQUIDITY).

## Watchdog (silent auto-recovery)

The watchdog (`watchdog_v2.py`) runs via cron every 15 minutes. It:
1. Checks container → restarts if down
2. Kills xmrig silently
3. Checks MT5 terminal → starts if not running
4. Checks RPyC port 8001 → starts if not listening
5. Checks MT5 login → restarts terminal if login fails
6. Checks bot process → starts if not running
7. **If all OK: silent (exit 0, no message sent)**
8. **If unfixable: sends alert to Telegram**

Cron job name: "ClawTrade Watchdog"

### ⚠️ CRITICAL: Never kill -9 wineserver/wineboot/winedevice

**Killing Wine system processes (wineserver, wineboot, winedevice) with `kill -9` PERMANENTLY corrupts the Wine prefix.** The prefix's kernel32.dll becomes unloadable (`c0000135` error) and cannot be fixed — the entire container + volume must be rebuilt from scratch.

**Safe to kill:** `terminal64.exe` (MT5 terminal), `python.exe` (mt5linux server), xmrig
**NEVER kill:** `wineserver`, `wineboot`, `winedevice`, `explorer.exe`

If Wine is stuck or MT5 terminal is unresponsive:
1. Kill ONLY `terminal64.exe`: `docker exec claw-trade-mt5 bash -c 'kill $(pgrep -f terminal64) 2>/dev/null'`
2. If that doesn't work, **restart the whole container**: `docker restart claw-trade-mt5`
3. NEVER reach for `kill -9` on Wine system processes as a "cleanup" step

### ⚠️ CRITICAL: iptables may block VNC port 3000

If the user cannot access VNC at `http://SERVER_IP:3000`, check iptables:
```bash
iptables -L INPUT -n | grep 3000
# If DROP rule exists:
iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
```
This DROP rule may exist from initial server setup and persists across reboots.

### ⚠️ USER PREFERENCES: Silent + AI-driven background operations

**1. NO unsolicited notifications** ("ผมไม่ต้องการให้ส่งมาหาผมมันน่าลำคาญ")
- `deliver: local` for all cron jobs — NOT `deliver: origin`
- Do NOT set `notify_on_complete=true` for long-running bot processes unless asked
- Only alert if auto-recovery FAILS

**2. User wants AI-driven watchdog, NOT script-only** ("ใช้aiรันจริงสิ")
- Do NOT use `no_agent: true` — the user wants AI to diagnose and auto-fix
- Use `enabled_toolsets: ["terminal"]` to limit token overhead
- The AI watchdog should run the script, check exit code, fix issues, re-verify, and output a one-line summary
- Do NOT send any message to the user unless recovery fails

**Correct cron setup:**
```
action: create
name: "ClawTrade AI Watchdog"
schedule: "*/15 * * * *"
deliver: local
enabled_toolsets: ["terminal"]
# NOT no_agent: true
```

### Cron script setup

The watchdog script must be placed in `~/.hermes/profiles/trader/scripts/` (the Hermes scripts directory), NOT in the project directory. The `cronjob` tool only resolves scripts relative to this directory.

```bash
# Copy watchdog to scripts directory
cp /root/Claw_Trade/watchdog_v2.py ~/.hermes/profiles/trader/scripts/clawtrade_watchdog.py
chmod +x ~/.hermes/profiles/trader/scripts/clawtrade_watchdog.py
```

When creating the cron job:
```
action: create
name: "ClawTrade Watchdog"
schedule: "*/15 * * * *"
script: "clawtrade_watchdog.py"   # filename only, not full path
no_agent: true                     # no LLM, no tokens
deliver: local                     # SILENT — do not send to user's chat
```

## Pitfalls

- **⚠️ NEVER `kill -9` wineserver/wineboot/winedevice**: Killing these processes mid-init corrupts the Wine prefix. The symptom is `wine: could not load kernel32.dll, status c0000135` — Wine becomes permanently broken for that prefix. The ONLY fix is deleting the prefix (or the entire `/config` volume) and recreating from scratch. If you need to stop MT5, kill `terminal64.exe` only, NOT the Wine infrastructure processes. If Wine seems stuck, `docker restart` the whole container rather than killing individual Wine processes.
- **xmrig returns after kill**: The s6 supervisor or a hidden process respawns it. Must `pkill -9 -f xmrig` AND `pkill -9 -f xmr_linux` AND `rm -rf /tmp/xmrig /tmp/xmr_linux_amd64`. The watchdog handles this every tick.
- **MT5 terminal crashes silently**: Wine output goes to `/tmp/mt5_wd.log` — check there if MT5 disappears.
- **`tz_convert()` error fetching historical data**: Known pandas version mismatch between host (3.x) and Wine Python (3.9). Bot falls back to Yahoo Finance cache (`mt5_historical_data.csv`) — not critical, just degraded data.
- **Container restart loses MT5 login**: After `docker restart`, MT5 terminal must be relaunched AND `mt5.initialize(login=..., password=..., server=...)` must be called with credentials from `.env`.
- **`wine tasklist` for process check**: Use `WINEPREFIX=/config/.wine wine tasklist` to list Wine processes. `ps aux` inside container won't show Wine processes by name. Must run as user `abc` (owner of `.wine`): `docker exec -u abc -e WINEPREFIX=/config/.wine claw-trade-mt5 bash -c 'wine tasklist 2>&1'`. Running as root gives `wine: '/config/.wine' is not owned by you`.
- **MT5 `Authorization failed` after bot kill/restart**: When the bot is killed and restarted, `mt5.initialize()` may fail with `Error code: (-6, 'Terminal: Authorization failed')`. Kill ALL terminal64.exe instances via `wine taskkill /f /im terminal64.exe`, wait 3s, relaunch one, wait 20s, then re-initialize.
- **Multiple `terminal64.exe` instances cause auth conflicts**: After restarts or manual launches, 2+ MT5 terminals may run. Always `taskkill` all instances before relaunching to ensure single session.
- **iptables may block VNC port 3000**: Even when VNC/nginx is running correctly inside the container, an iptables DROP rule on the host can block external access. Check `iptables -L INPUT -n | grep 3000` — if you see DROP before ACCEPT, fix with `iptables -I INPUT -p tcp --dport 3000 -j ACCEPT`. This is a common cause of "can't access VNC" that isn't a VNC problem at all.
- **RPyC version mismatch causes `ValueError: not enough values to unpack`**: The `start.sh` script downgrades rpyc to 5.2.3 inside Wine Python (from 6.0.2 shipped by the image). The host-side rpyc must also be 5.2.3. If you see this error, check `python3 -c "import rpyc; print(rpyc.__version__)"` on host vs `wine python -c "import rpyc; print(rpyc.__version__)"` in container — they must match.
- **Fresh MT5 install does NOT auto-login**: After a fresh container/volume, `start.sh` installs MT5 but the terminal shows "Application could not be started, or no application associated with the specified file." MT5 terminal must be launched separately AND the user must log in manually via VNC at `http://VPS-IP:3000` using credentials from `.env`. The `mt5.initialize()` call will time out (30s+) until the terminal has completed broker login.
- **numpy fix must be re-applied after fresh install**: The Wine-side `metatrader5.py` at `/config/.wine/drive_c/Program Files (x86)/Python39-32/Lib/site-packages/mt5linux/metatrader5.py` gets overwritten by `start.sh`'s pip install. Must re-add `self.__conn.execute("import numpy as np")` after line 20 (`self.__conn.execute("import datetime")`) every time the container/volume is recreated.

## VNC Access

- VNC web UI: `http://VPS-IP:3000` (KasmVNC via nginx)
- VNC requires port 3000 open in iptables: `iptables -I INPUT -p tcp --dport 3000 -j ACCEPT`
- Check with: `iptables -L INPUT -n | grep 3000` — if a DROP rule exists above the ACCEPT, VNC won't be reachable externally even though it's running
- Screenshot the VNC display with: `docker exec claw-trade-mt5 bash -c 'DISPLAY=:1 ffmpeg -y -f x11grab -video_size 1522x702 -i :1 -frames 1 /tmp/screen.png 2>&1'` then `docker cp claw-trade-mt5:/tmp/screen.png /tmp/mt5_screen.png` — note screen size can vary, check with `xdpyinfo -display :1` or inspect the error message for the actual dimensions

## How to verify MT5 is logged in (without waiting for mt5.initialize timeout)

`mt5.initialize()` hangs for 30s+ if MT5 terminal hasn't completed broker login. Check login status FIRST:

```bash
# Best method: wmctrl shows window title with login status
docker exec claw-trade-mt5 bash -c 'apt-get install -y -qq wmctrl 2>/dev/null; DISPLAY=:1 wmctrl -l'
# Logged in: "106123714 - FBSTradestone-Demo: Demo Account - Hedge - Tradestone Limited"
# Not logged in: no output, or different title

# Alternative: xwininfo tree
docker exec claw-trade-mt5 bash -c 'DISPLAY=:1 xwininfo -tree -root 2>/dev/null | grep terminal64'
```

**xdotool will NOT find MT5 windows** even when they exist. Use `wmctrl -l` or `xwininfo -tree -root` instead.

## Useful Commands

```bash
# Check all layers at once
python3 /root/Claw_Trade/watchdog_v2.py

# Manual MT5 connection test
python3 -c "
import sys; sys.path.insert(0,'/root/Claw_Trade/src')
import mt5linux, os
from dotenv import load_dotenv
load_dotenv('/root/Claw_Trade/.env')
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
mt5.initialize(login=int(os.getenv('MT5_LOGIN')), password=os.getenv('MT5_PASSWORD'), server=os.getenv('MT5_SERVER'))
print('Terminal:', mt5.terminal_info())
print('Account:', mt5.account_info())
"

# View current positions
cat /root/Claw_Trade/live_positions.json | python3 -m json.tool

# Bot log tail
tail -30 /tmp/claw_live5.log
```

## References

- `references/mt5linux-rpyc-quirks.md` — detailed notes on mt5linux RPyC integration issues and fixes
- `references/startup-troubleshooting.md` — step-by-step diagnostic for common startup failures
- `references/container-rebuild-procedure.md` — full container+volume rebuild when Wine prefix is corrupted (last resort)