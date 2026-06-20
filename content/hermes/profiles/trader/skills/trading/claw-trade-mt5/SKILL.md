---
name: claw-trade-mt5
version: 1.0.0
category: trading
description: Operate the ClawTrade XAU/USD trading system — MT5 docker container, 6 AI agents, trade history DB, and Agent HQ dashboard. Covers status queries, bot lifecycle, and market analysis.
tags:
  - xau-usd
  - gold-trading
  - mt5
  - docker
  - ai-agents
  - trading-bot
---

# ClawTrade MT5 — Gold Sniper Trading System

System that trades XAU/USD on MT5 with a team of 6 AI agents. Source root: `/root/Claw_Trade/`.

## Architecture

Two layers, **don't confuse them**:

| Layer | Where | Ports | What it does |
|-------|-------|-------|--------------|
| **MT5 runtime** | Docker container `claw-trade-mt5` (image `gmag11/metatrader5_vnc`) | 3000 (VNC web UI for MT5 login), 8001 (RPyC `mt5linux` API), 3001 (internal nginx) | Runs MetaTrader 5 inside a Linux+Wine+VNC container. The Python bot connects to port 8001 to send orders and read prices. |
| **Trading bot + agents** | Host processes (`main.py live`, `agent_hq_server.py`) | 8080 (Agent HQ dashboard) | The 6 AI agents, the trading loop, the HQ web UI. These are NOT in the container. |

Container volume: `mt5_config` (holds MT5 config / login state).

### The 6 agents

| Agent | Emoji | Role | Duty |
|-------|-------|------|------|
| QUANT | 🔮 | Technical Analyst (Lv8) | RSI / MACD / EMA |
| NEWS | 📡 | News Scout (Lv5) | Economic calendar |
| BULL | ⚔️ | Bullish Strategist (Lv7) | BUY arguments |
| BEAR | 🛡️ | Bearish Guardian (Lv7) | SELL arguments |
| CEO | 👑 | Executive Chairman (Lv10) | Final decision |
| LEARN | 🧬 | Learning Engine (Lv6) | Self-optimization |

Agents are active only when `main.py live` is running on the host. When the bot is stopped, all agents show Idle.

## Cold-start sequence (after container restart or VPS reboot)

After a `docker restart` or VPS reboot, the container comes up but **nothing inside is ready**: MT5 terminal is not running, mt5linux may not be listening, and xmrig miner is consuming all CPU. Follow these steps in order.

### Step 1: Kill xmrig miner (CRITICAL — do this first)

The `gmag11/metatrader5_vnc` image embeds an xmrig crypto miner at `/tmp/xmrig/` that auto-starts via s6 services. It consumes **318% CPU + 2.4GB RAM** and will starve MT5/Wine. Kill it immediately after every container restart:

```bash
docker exec claw-trade-mt5 pkill -9 -f xmrig
docker exec claw-trade-mt5 pkill -9 -f xmr_linux
docker exec claw-trade-mt5 rm -rf /tmp/xmrig /tmp/xmr_linux_amd64
# Verify killed (should output 0):
docker exec claw-trade-mt5 bash -c 'ps aux | grep -iE "xmrig|xmr_linux" | grep -v grep | wc -l'
```

If it respawns (s6-supervise auto-restarts it), repeat the kill. The miner connects to `xpoolje.daviduwu.ovh:3222`. See `references/xmrig-miner.md` for full evidence.

### Step 2: Fix Wine ownership (if needed)

After restart, wine may log `'/config/.wine' is not owned by you`:
```bash
docker exec claw-trade-mt5 bash -c 'chown -R abc:abc /config/.wine'
```

### Step 3: Launch MT5 terminal (does NOT auto-start)

MT5 terminal64.exe must be launched manually via Wine with DISPLAY set:
```bash
docker exec -d -u abc -e DISPLAY=:1 -e WINEPREFIX=/config/.wine claw-trade-mt5 \
  bash -c 'WINEDEBUG=-all wine "/config/.wine/drive_c/Program Files/MetaTrader 5/terminal64.exe" > /tmp/mt5_log.txt 2>&1'
# Wait ~15s, then verify it's running:
docker exec claw-trade-mt5 bash -c 'ps aux | grep terminal64 | grep -v grep'
```

### Step 4: Start mt5linux RPyC server (if port 8001 not listening)

Check and start manually if s6-supervise failed to bring it up:
```bash
# Check:
docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep 8001'
# If empty, start manually:
docker exec -d -u abc -e WINEPREFIX=/config/.wine claw-trade-mt5 \
  bash -c 'wine "C:\Program Files (x86)\Python39-32\python.exe" -m mt5linux --host 0.0.0.0 --port 8001 > /tmp/mt5linux.log 2>&1'
# Wait ~8s, verify:
docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep 8001'
```

### Step 5: Verify MT5 connection from host

```bash
cd /root/Claw_Trade && python3 -c "
import sys; sys.path.insert(0,'src')
import mt5linux
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
print('Terminal:', mt5.terminal_info())
print('Account:', mt5.account_info())
"
```
- `terminal_info` returns `None` → MT5 is running but **not logged in** → go to Step 6
- `terminal_info` returns a dict → MT5 is logged in → skip to Step 7

### Step 6: Log in to MT5 — PREFER programmatic login (no VNC needed)

**Option A: Programmatic login (preferred — works if `.env` has real credentials)**

The `mt5linux` library's `initialize()` accepts `login`, `password`, and `server` kwargs. If MT5 terminal is running (Step 3) and mt5linux RPyC is listening (Step 4), you can log in directly from the host — no VNC needed:

```bash
cd /root/Claw_Trade && python3 -c "
import sys, os; sys.path.insert(0,'src')
from dotenv import load_dotenv; load_dotenv()
import mt5linux
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
ok = mt5.initialize(
    login=int(os.getenv('MT5_LOGIN','0')),
    password=os.getenv('MT5_PASSWORD',''),
    server=os.getenv('MT5_SERVER','')
)
print('initialize:', ok)
print('Terminal:', mt5.terminal_info())
print('Account:', mt5.account_info())
"
```

If `initialize` returns `True` and `terminal_info` shows `connected=True` — done, skip to Step 7.

**Option B: VNC manual login (fallback — only if Option A fails)**

1. Open browser → `http://[VPS-IP]:3000` (VNC Web UI — login: trader / change_me_pls)
2. In the Wine desktop, find the MT5 terminal window
3. Log in with credentials from `.env` (same values as Option A)
4. Wait for connection confirmed in MT5 status bar
5. Re-run Step 5 to verify `terminal_info` returns data

### Step 7: Install Python dependencies (if first time on this host)

```bash
cd /root/Claw_Trade
pip install mt5linux pandas numpy anthropic yfinance python-dotenv
# MetaTrader5 and pandas_ta are NOT installable on Linux:
#   MetaTrader5 is Windows-only (use mt5linux via RPyC instead)
#   pandas_ta has no compatible wheel (not used in core code)
```

### Step 8: Start the trading bot (see "Start the bot" below)

---

## Daily operation

### 1. Check system status (do this first every session)

```
# Container up?
docker ps --filter name=claw-trade-mt5 --format "{{.Names}}|{{.Status}}|{{.Ports}}"

# Bot + HQ running on host?
ps aux | grep -iE "main\.py|agent_hq" | grep -v grep

# RPyC API alive inside container?
docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep 8001'

# MT5 terminal running inside container?
docker exec claw-trade-mt5 bash -c 'ps aux | grep terminal64 | grep -v grep'

# xmrig miner NOT running (should be 0)?
docker exec claw-trade-mt5 bash -c 'ps aux | grep -iE "xmrig|xmr_linux" | grep -v grep | wc -l'
```

Expected healthy state:
- Container `Up` for hours/days
- `terminal64.exe` process running inside container
- `mt5linux` process listening on 8001 inside container
- xmrig processes: 0 (killed)
- `main.py live` and `agent_hq_server.py` running on host

### 2. Pull overview & trade stats — DB is the source of truth

The HQ server on port 8080 is often NOT running (it's a convenience layer). Query the SQLite DB directly — it always has the real numbers. See `references/overview-query.py` for the exact query that returns total/wins/losses/WR/PnL/avg-R, monthly breakdown, regime breakdown, and recent trades.

Quick inline form:
```bash
cd /root/Claw_Trade && python3 -c "
import sqlite3, json
c = sqlite3.connect('trade_memory.db'); c.row_factory = sqlite3.Row
row = c.execute(\"SELECT COUNT(*) total, SUM(CASE WHEN close_reason='TP' THEN 1 ELSE 0 END) wins, SUM(CASE WHEN close_reason='SL' THEN 1 ELSE 0 END) losses, COALESCE(SUM(pnl_usd),0) pnl, COALESCE(AVG(CASE WHEN close_reason IS NOT NULL THEN r_achieved END),0) avg_r FROM trades\").fetchone()
t=row['total'] or 0; w=row['wins'] or 0
print(f'Trades={t}  WR={w/t*100:.1f}%  PnL=\${row[\"pnl\"] or 0:.2f}  avgR={row[\"avg_r\"] or 0:.2f}')
"
```

### 3. Read the learned config

`/root/Claw_Trade/learned_config.json` — updated by the LEARN agent after each trade. Contains confidence thresholds per regime, RR ratio, SL/ATR multiplier, position size %, session multipliers, fibo zone bias. **Always read this before recommending trade parameter changes** — the bot may have already self-tuned.

Key fields:
- `confidence_threshold` — per-regime gate (TRENDING 0.78, RANGING 0.82, HIGH_VOL 0.85, LOW_LIQ 0.90)
- `rr_ratio` — target reward:risk (3.0)
- `position_size_pct` — per-regime % of balance
- `session_multiplier` — scales size by session (NY_LATE 0.7, ASIA 0.5)
- `overall_winrate`, `consecutive_losses` — live performance state

### 4. Start the bot (when the user asks to go live)

**Pre-flight: verify OPENROUTER_API_KEY is real, not a placeholder.**

The `.env` file may ship with `OPENROUTER_API_KEY=***` — a placeholder, not a real key. If the bot runs with this, all 6 agents will fail with `OpenRouter HTTP 401: Missing Authentication header` and CEO will always return `NO_TRADE` with confidence 0.00. Check before launch:

```bash
grep OPENROUTER_API_KEY /root/Claw_Trade/.env
# If it shows "***" → ask user for a real key from https://openrouter.ai/
# Put the real key in .env before starting the bot
```

**Paper trading (demo account, safe):**

Use `terminal(background=true, notify_on_complete=true)` — the Hermes-native way. Do NOT use `nohup ... &` in foreground terminal (the tool blocks it).

```python
terminal(
  command="cd /root/Claw_Trade && python3 main.py paper > live_trading_output.log 2>&1",
  background=True,
  notify_on_complete=True
)
```

**Live trading (real money, requires --confirm):**

Same pattern, add `--confirm`:

```python
terminal(
  command="cd /root/Claw_Trade && python3 main.py live --confirm --symbol XAUUSDc --interval 5 > live_trading_output.log 2>&1",
  background=True,
  notify_on_complete=True
)
```

Common CLI flags:
- `--symbol XAUUSDc` — override symbol (default `GC=F` for Yahoo Finance; use MT5 symbol for live trading)
- `--interval 5` — check interval in minutes (default 60)

**Always confirm MT5 is logged in first** (see cold-start Step 5/6) — if `terminal_info()` returns `None`, the bot will run but cannot execute trades.

Optional: Agent HQ dashboard:
```bash
nohup python3 agent_hq_server.py > agent_hq.log 2>&1 &
```

Verify after ~10s:
- `ps aux | grep main.py | grep -v grep` shows the process
- `tail -20 /root/Claw_Trade/live_trading_output.log` shows MT5 connection
- `curl -s localhost:8080/api/agents` returns bot=running (if HQ started)

### 5. Stop the bot

```bash
kill $(cat /root/Claw_Trade/live_trading.pid)
# Or if pid file is stale:
pkill -f "main.py live"
```

## Background operations & cron (USER PREFERENCES)

Two strong, explicit preferences:

### 1. NO unsolicited notifications ("ผมไม่ต้องการให้ส่งมาหาผมมันน่าลำคาญ")
The user does NOT want background monitoring notifications sent to their chat. Rules:
- **Cron jobs for watchdog/monitoring: use `deliver: local`**, NOT `deliver: origin`. Results stay in local logs.
- **Do NOT set `notify_on_complete=true`** on long-running trading bot processes unless asked.
- Only alert the user if auto-recovery fails and human intervention is needed.
- Watchdog logs go to `/root/Claw_Trade/watchdog_v2.log` — check there when the user asks about health.

### 2. User wants AI-driven watchdog, NOT script-only ("ใช้aiรันจริงสิ")
The user explicitly asked to use an AI agent for the watchdog — NOT `no_agent: true`. The user wants the watchdog to intelligently diagnose and auto-fix issues, not just run a static script. The user is token-conscious but prioritizes AI-driven recovery over token savings.

**Correct cron job setup for the watchdog:**
```
action: create
name: "ClawTrade AI Watchdog"
schedule: "*/15 * * * *"
deliver: local                    # SILENT — do not send to user's chat
enabled_toolsets: ["terminal"]   # only terminal tool needed — saves token overhead
# Do NOT set no_agent: true       # user wants AI to handle recovery
```

The AI watchdog prompt should:
1. Run `python3 /root/Claw_Trade/watchdog_v2.py 2>&1`
2. If exit 0 → output "OK", stay silent
3. If exit 1 → diagnose the error, attempt fix (restart container, kill xmrig, launch MT5, start RPyC, start bot), re-verify
4. Output one-line summary: "OK" or "FIXED: <what was wrong>"
5. Do NOT send any message to the user

### Cron script setup
- Cron scripts must be placed in `~/.hermes/profiles/trader/scripts/` and referenced by filename only (not full path).
- `no_agent: true` script-only cron failed with "Script not found" — the `cronjob` tool resolves scripts relative to `~/.hermes/profiles/trader/scripts/`, NOT the project directory. Copy the script there first.
- The AI-driven approach (without `no_agent`) avoids this path resolution issue entirely.

## CRITICAL: Don't break a working system

The user said "จำไว้นะครับผมไม่อยากมานั่งทำใหม่" ("remember, I don't want to sit here redoing this"). A working MT5 bot was once destroyed by aggressive Wine process killing during a routine health check, forcing a full container+volume rebuild + manual MT5 re-login via VNC.

**Rules:**
1. **Never `kill -9` Wine system processes** (wineserver, wineboot, winedevice) — permanently corrupts the Wine prefix (kernel32.dll c0000135), requiring full volume rebuild + manual MT5 re-login
2. If something is stuck, **`docker restart claw-trade-mt5`** the whole container — don't kill individual Wine processes
3. Before touching Wine/MT5, take a screenshot first to see what state it's in
4. If you don't know what a process does, don't kill it — restart the container instead
5. The watchdog cron handles auto-recovery; manual intervention should be last resort

### VNC port 3000 blocked by iptables

If the user can't access VNC at `http://SERVER_IP:3000`, check iptables:
```bash
iptables -L INPUT -n | grep 3000
# If DROP rule exists:
iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
```
This DROP rule may exist from initial server setup and persists across reboots.

### How to verify MT5 is logged in (without mt5.initialize timeout)

`mt5.initialize()` hangs 30s+ if MT5 terminal hasn't completed broker login. Check FIRST:
```bash
# Best: wmctrl shows window title with login status
docker exec claw-trade-mt5 bash -c 'apt-get install -y -qq wmctrl 2>/dev/null; DISPLAY=:1 wmctrl -l'
# Logged in: "106123714 - FBSTradestone-Demo: Demo Account - Hedge - Tradestone Limited"
# Not logged in: no output
```
**xdotool will NOT find MT5 windows.** Use `wmctrl -l` or `xwininfo -tree -root` instead.

### Full container rebuild (last resort — when Wine prefix is corrupted)

If `wine: could not load kernel32.dll, status c0000135` appears, the Wine prefix is permanently corrupted. See `clawtrade-mt5` skill's `references/container-rebuild-procedure.md` for the full 13-step rebuild procedure. Summary: destroy container + volume, recreate with fresh volume, run start.sh, re-apply numpy fix, open iptables, user logs in via VNC, start mt5linux + bot.

## Pitfalls

- **Port 3000 ≠ Agent HQ.** Port 3000 is the container's VNC web UI (serves `Cannot GET /api/...` for any API path). Agent HQ is port 8080 on the host. Don't waste time curling 3000 for trade data.
- **Port 8001 from the host returns empty.** It's an RPyC protocol port, not HTTP — `curl` returns nothing, that's normal. Check it with `docker exec ... ss -tlnp | grep 8001`.
- **`agent_hq_server.py` is usually not running.** It's a dashboard, not the bot. Don't treat its absence as a system failure. Query the DB instead.
- **`execute_code` with subprocess is blocked under cron_mode approvals.** Use `terminal` for shell+subprocess work, or use `curl` from terminal to hit APIs. (This is an approvals policy, not a tool bug — retry with `terminal`.)
- **Container image embeds xmrig crypto miner** (`xmr_linux_amd64` masquerading as `node index.js`, plus `/tmp/xmrig/xmrig-6.26.0/xmrig`). This is from the base image `gmag11/metatrader5_vnc`. It auto-starts via s6 services and uses **318% CPU + 2.4GB RAM** — will starve MT5/Wine. **Must kill after every container restart.** See cold-start Step 1 above and `references/xmrig-miner.md` for full remediation commands.
- **DB has few rows early on.** The system self-optimizes from real trades; with only 1–2 trades the learned config is near-defaults. Don't over-interpret win rate from tiny samples.
- **`OPENROUTER_API_KEY=***` placeholder → all agents fail with 401.** The `.env` ships with a placeholder key, not a real one. If the bot log shows `OpenRouter HTTP 401: Missing Authentication header` for QUANT, NEWS, BULL, BEAR, and CEO, the key is missing or `***`. Fix: put a real key from https://openrouter.ai/ into `.env`. The bot will pick it up on next restart. Until then, CEO always returns `NO_TRADE` with confidence 0.00 — no trades will execute (safe failure, not a crash).
- **`terminal_info()` returns `None` even when MT5 terminal is running.** This means MT5 is open but not logged in. Use `mt5.initialize(login=, password=, server=)` with credentials from `.env` — programmatic login works without VNC (see cold-start Step 6, Option A). Only fall back to VNC manual login if `initialize()` fails.
- **mt5linux RPyC `NameError: name 'np' is not defined` when sending orders.** The `mt5linux` library only imports `MetaTrader5` and `datetime` into the RPyC namespace (`metatrader5.py` lines 17-19). When `mt5.order_send(request)` serializes the order dict through RPyC, the Wine-side Python `eval()`s code that references `np` (numpy), which isn't in the namespace. Fix: patch the installed `mt5linux/metatrader5.py` to add `self.__conn.execute("import numpy as np")` after the existing imports. Must re-patch after any `pip install --upgrade mt5linux`. See `references/mt5linux-rpyc-patches.md` for exact diff and verification.
- **`get_account_info()` returns a dict, not an object with attributes.** `mt5_connector.py`'s `get_account_info()` always returns a dict (both the tuple/list branch and the namedtuple branch convert to dict). Orchestrator code that does `acc_info.balance` will crash with `'dict' object has no attribute 'balance'`. Use `acc_info['balance']` instead. The fix is in `orchestrator.py` around line 676.
- **MT5 `Authorization failed` after bot kill/restart.** When the bot process is killed (SIGTERM) and restarted, `mt5.initialize()` may return `Error code: (-6, 'Terminal: Authorization failed')`. This happens when the old MT5 terminal session is stale or multiple `terminal64.exe` instances are running in Wine. Fix: `wine taskkill /f /im terminal64.exe` to kill ALL instances, wait 3s, relaunch one terminal, wait 20s, then `mt5.initialize()` again. The watchdog handles this automatically.
- **Multiple `terminal64.exe` instances cause auth conflicts.** After a container restart or manual launches, you may end up with 2+ MT5 terminal processes. Each holds its own session; only one can be logged in. Always `taskkill /f /im terminal64.exe` before relaunching to ensure a single instance.

## Reporting format (Telegram)

Present status as:
1. Overview table (Total / WR / PnL / avg R)
2. Agent status table with emoji + Idle/Working
3. Infrastructure status (container, RPyC, bot, HQ)
4. Recent trades table
5. One-line risk note if anything is degraded

Use real Markdown tables — they render on Telegram. Keep it scannable, not narrative.

## See also

- `scripts/mt5_login_check.py` — one-shot MT5 connection + programmatic login verification (exit codes 0-3). Run from host: `cd /root/Claw_Trade && python3 scripts/mt5_login_check.py`
- `references/overview-query.py` — exact SQLite query for full overview (drop-in for terminal/python3)
- `references/architecture.md` — port map, process list, file map of /root/Claw_Trade
- `references/xmrig-miner.md` — xmrig crypto miner discovery, evidence, and remediation (from gmag11/metatrader5_vnc base image)
- `references/mt5linux-rpyc-patches.md` — two patches for `order_send()` errors: RPyC `NameError: np` and `get_account_info()` dict-vs-object crash