---
name: claw-trade-linux-migration
description: 'Complete workflow: migrate Claw_Trade (or any MT5-based Python trading bot) to Linux. Covers Docker+mt5linux (free), MetaAPI.cloud (paid), and Wine approaches with actual tradeoffs.'
category: trading
---

# Claw_Trade Linux Migration

Run any MT5 Python bot on Linux — no Windows VPS needed.

## User Context
- **User**: TK TF (Thai speaker)
- **Broker**: FBS (MT5 account)
- **Project**: [Claw_Trade](https://github.com/michoder26-cloud/Claw_Trade) — XAU/USD MMTC v2.0 multi-agent AI trading system
- **Repo**: `/root/Claw_Trade`
- **Final approach used**: Docker + mt5linux (RPyC bridge)

## Architecture

```
┌─────────── Linux ───────────┐     RPyC (port 8001)    ┌──────────── Docker ──────────┐
│  Claw_Trade (Python 3.12)   │ ◄────────────────────►  │  Wine + MT5 + Python 3.9     │
│  from mt5linux import       │                         │  Logged into FBS broker      │
│      MetaTrader5 as mt5     │                         │  linuxserver/kasmvnc base    │
└─────────────────────────────┘                         └──────────────────────────────┘
```

## Three Approaches — Tradeoffs

| Approach | Cost | Setup | Stability | Latency |
|:---------|:----:|:-----:|:---------:|:-------:|
| **🥇 Docker + mt5linux** | Free | ⭐⭐ Moderate | Medium | Low (LAN RPyC) |
| **🥈 MetaAPI.cloud** | $30/mo | ⭐ Easy | High | Very low (cloud) |
| **🥉 Wine native** | Free | ⭐⭐⭐ Hard | Low | Native |

**This skill documents approach 1 (Docker + mt5linux) since it's what was implemented.**

## Files Created/Modified

| File | Purpose |
|------|---------|
| `src/mt5_connector.py` | **Modified** — added mt5linux support (auto-detect) |
| `src/orchestrator.py` | **Reverted** — back to MT5Connector() (no factory) |
| `src/metaapi_connector.py` | **Archived** — not used after Docker pivot |
| `docker-compose.yml` | **NEW** — runs gmag11/metatrader5_vnc container |
| `optimized_config.json` | Safer parameters from backtest analysis |

## Setup Steps

### 0. Prerequisites
```bash
apt install docker.io docker-compose-v2
# Or: curl -fsSL https://get.docker.com | bash
```

### 1. Pull & Run MT5 Docker Container
```bash
docker pull gmag11/metatrader5_vnc    # ~4-6GB, one-time download
docker run -d \
  --name claw-trade-mt5 \
  --restart unless-stopped \
  -p 3000:3000 \
  -p 8001:8001 \
  -v mt5_config:/config \
  gmag11/metatrader5_vnc
```

### 2. Login to FBS via VNC (one-time manual step)
1. Open `http://<server-ip>:3000` in a browser
2. Login: `abc` / `abc` (default KasmVNC credentials)
3. On the virtual desktop, launch **MetaTrader 5**
4. **File → Login to Trade Account** → enter FBS credentials
5. (Optional) Stop the VNC session — it's only needed for initial login

### 3. Fix numpy + Wine compatibility (CRITICAL)
The bundled MetaTrader5 `_core.pyd` needs numpy 1.x (1.20-1.26), NOT 2.x.
The container auto-install may get this wrong.

```bash
# Install cabextract + vcrun2019 (needed for .pyd DLL loading)
docker exec claw-trade-mt5 apt-get install -y cabextract
docker exec claw-trade-mt5 wget -q https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks -O /usr/local/bin/winetricks
docker exec claw-trade-mt5 chmod +x /usr/local/bin/winetricks
docker exec claw-trade-mt5 chown -R abc:abc /config/.wine
docker exec --user abc claw-trade-mt5 bash -c 'export WINEPREFIX=/config/.wine && winetricks -q vcrun2019'

# Fix numpy version to match MetaTrader5 ABI
docker exec --user abc claw-trade-mt5 bash -c '
  export WINEPREFIX=/config/.wine
  wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -m pip install numpy==1.24.3 --force-reinstall
'

# Verify import works inside Wine
docker exec --user abc claw-trade-mt5 bash -c '
  export WINEPREFIX=/config/.wine
  wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -c "
import MetaTrader5 as mt5
print(\"Import OK:\", mt5.initialize())
print(\"Version:\", mt5.version())
mt5.shutdown()
"
'
```

### 4. Start mt5linux Server
```bash
docker exec --user abc claw-trade-mt5 bash -c '
  export WINEPREFIX=/config/.wine
  wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -m mt5linux --host 0.0.0.0 --port 8001
'
```
Run this in background (`docker exec -d` variant or use terminal background=true). Verify with: `python3 -c "import socket; s=socket.socket(); s.settimeout(3); s.connect(('localhost',8001)); print('OK'); s.close()"`

### 4. Configure Environment
```bash
# /root/Claw_Trade/.env
MT5_HOST=127.0.0.1        # Docker host
MT5_PORT=8001             # RPyC port (8337 for pymt5linux)
MT5_LOGIN=your_fbs_account
MT5_PASSWORD=your_fbs_password
MT5_SERVER=FBS-Demo       # or FBS-Real
MT5_SYMBOL=XAUUSD
DATA_SOURCE=yfinance       # for backtesting (works on Linux)
```

### 5. Validate Connection
```bash
cd /root/Claw_Trade
source .venv/bin/activate

# Quick port check
python3 -c "import socket; s=socket.socket(); s.settimeout(3); s.connect(('127.0.0.1',8001)); print('Port OK'); s.close()"

# Full verification (see references/docker-mt5linux-setup.md for details)
python3 scripts/verify_mt5linux.py
```

### 6. Run the Bot
```bash
# Backtest (no Docker needed)
USE_MOCK_AI=true python main.py backtest --interval 1h --start-date 2026-05-01 --end-date 2026-05-31

# Paper trading (needs Docker MT5 running + logged in)
python main.py paper --interval 60

# Live (needs Docker + real FBS account)
python main.py live --confirm --interval 60
```

## How mt5_connector.py Auto-Detects

```python
# Order of priority in mt5_connector.py:
try:
    from mt5linux import MetaTrader5 as _MT5Client
    mt5 = _MT5Client(host=MT5_HOST, port=MT5_PORT)   # Linux + Docker
except ImportError:
    import MetaTrader5 as mt5                          # Windows native
```

The `mt5linux` MetaTrader5 class is a **drop-in proxy** — all `mt5.initialize()`, `mt5.login()`, `mt5.order_send()`, `mt5.symbol_info_tick()`, `mt5.copy_rates_range()` etc. work identically to the native MetaTrader5 package.

## Strategy Issues Found (May 2026 Backtest)

| Config | Win Rate | Net PnL | Problem |
|--------|----------|---------|---------|
| Default (6% risk) | 0% | -$6,518 | Lot x3 kills account |
| Safe (2% risk) | 0% | -$2,789 | Still loses, less damage |

**Root Cause**: The MMTC v2.0 "Tier 3 — Market Maker All-In" zone keeps triggering BUY with `[OVERRIDE_LOT_MULTIPLIER=3.0]` in a downtrending market. The tight SL ($3) is too narrow for 1-hour candles — SL gets hit within the next candle.

### Recommended Fixes (in `src/agents.py`)
1. **Trend filter**: Skip BUY if `close < ema_200` (downtrend confirmation)
2. **Cap lot multiplier**: Max 1.5x, never 3x
3. **Dynamic SL**: Use `max($3, 1.5 × ATR)` instead of fixed $3
4. **Trailing stop**: Enable on all trades (not just Tier 3)
5. **Regime guard**: Only allow Tier 3 entries in sideways/ranging regimes, skip in trending

## Pitfalls
- **First run takes ~5min**: Container auto-installs MT5 + Python on first launch; be patient.
- **VNC built into container**: The MT5 container (`gmag11/metatrader5_vnc`) includes KasmVNC **running on port 3000**. This VNC is part of the container, not a separate service — stopping VNC processes on the host will either have no effect or may destabilize the container. **Do not kill VNC on the host**; it's managed by the container.
- **VNC login needed once**: MT5 requires interactive login on first use. After that, MT5 stays logged in through container restarts. Use `http://<server-ip>:3000` to access the desktop.
- **mt5linux server auto-start fails**: If MT5 isn't logged in yet, the RPyC server won't start. Solution: log in via VNC, then start the server manually.
- **Docker restart persistence**: The `mt5_config` volume preserves MT5 login state across container restarts.
## 🔴 CRITICAL SECURITY: VNC built into container on port 3000

The `gmag11/metatrader5_vnc` image launches Xvnc with `-SecurityTypes None -disableBasicAuth -AlwaysShared`. This means **anyone who knows the URL can access the MT5 desktop without any password**. The web interface at port 3000 loads directly into the VNC session with no login screen. **Mitigation**: 
- Immediately block port 3000 on the host firewall (`iptables -A INPUT -p tcp --dport 3000 -j DROP`)
- Or use SSH tunnel instead of direct exposure
- Or switch to NoMachine (port 4000) which has proper authentication
- **Do not kill VNC processes on the host** — VNC runs inside the container; killing host VNC processes has no effect or may destabilize the container
- The bot does NOT need port 3000 to function — it trades via mt5linux API (port 8001)

## 🔴 CRITICAL SECURITY: Container Compromise — xmrig miner malware

**Real-world incident (June 2026):** An exposed port 3000 (VNC) led to an attacker gaining root access inside the container. The attacker installed xmrig cryptocurrency miner at `/tmp/xmrig/xmrig-6.26.0/xmrig` with `rwxrwxrwx` permissions and a `config.json` pool config. This miner consumed **640% CPU** and **2.4GB RAM**, causing OOM kills that brought down ALL Hermes agent gateways simultaneously.

**Detection signs:**
- `dmesg` shows `Out of memory: Killed process ... (xmrig)`
- Load average spikes to 50-99
- `find /var/lib/docker -name xmrig` finds miner binary inside container overlay
- Container filesystem shows `/tmp/xmrig/` with timestamp newer than container creation

**Immediate response:**
```bash
# Stop the compromised container NOW
docker stop claw-trade-mt5

# Verify miner is dead
pgrep -fa xmrig | grep -v grep  # should be empty

# Check memory freed
free -h  # should show ~8-9GB free (was ~4GB)

# If you MUST restart container, always:
# 1. Block port 3000 first
iptables -A INPUT -p tcp --dport 3000 -j DROP
# 2. Limit container resources
docker update --cpus=4 --memory=4g --memory-swap=4g claw-trade-mt5
```

**Prevention:**
- **NEVER expose port 3000 to internet** — always firewall-block or bind to localhost
- Regularly scan container for unauthorized binaries: `docker exec claw-trade-mt5 find /tmp /var /opt -type f -name "*rig*" 2>/dev/null`
- Monitor unexpected load spikes: if load > 20, check for miners immediately
- Use `--cpus` and `--memory` limits on ALL containers

## Container Management
```bash
# Start/Stop
docker start claw-trade-mt5
docker stop claw-trade-mt5

# Logs
docker logs claw-trade-mt5 -f

# Exec commands inside
docker exec claw-trade-mt5 wine python -c "import MetaTrader5; print('ok')"

# Remove and recreate (keeps config volume)
docker rm -f claw-trade-mt5
docker run -d --name claw-trade-mt5 ... (same args as above)
```