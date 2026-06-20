# ClawTrade Architecture Reference

## Port map

| Port | Host/Container | Service | Notes |
|------|----------------|---------|-------|
| 3000 | container | VNC web UI (nginx) | For logging into MT5 the first time. NOT an API. `Cannot GET /api/...` is expected. |
| 3001 | container | nginx (internal) | No direct use from host. |
| 8001 | container | RPyC `mt5linux` | Python bot connects here. RPyC protocol — `curl` returns empty, that's normal. Verify with `docker exec ... ss -tlnp \| grep 8001`. |
| 8080 | host | `agent_hq_server.py` | Dashboard + `/api/overview` + `/api/agents`. Usually NOT running. DB is the fallback. |

## Key processes

### Inside container
- `Xvnc :1` — virtual display for MT5 (`DISPLAY=:1`)
- `nginx` — serves VNC web UI on port 3000
- `terminal64.exe` (via Wine) — **MT5 terminal itself. Does NOT auto-start.** Must launch manually after container restart. Path: `/config/.wine/drive_c/Program Files/MetaTrader 5/terminal64.exe`
- `python -m mt5linux --host 0.0.0.0 --port 8001` (via Wine, Python39-32) — the RPyC bridge to MT5. Sometimes fails to auto-start via s6-supervise; start manually if port 8001 is not listening.
- `xmr_linux_amd64` (disguised as `node index.js`) + `/tmp/xmrig/xmrig-6.26.0/xmrig` — **XMR miner from base image, uses 318% CPU + 2.4GB RAM.** Must kill after every container restart. See `references/xmrig-miner.md` for full remediation.

### On host
- `python3 main.py live` — the trading bot + 6 agents. Write PID to `live_trading.pid`.
- `python3 agent_hq_server.py` — optional dashboard on :8080
- `watchdog.py` — monitors container, restarts if it dies

## File map of /root/Claw_Trade

| File | Purpose |
|------|---------|
| `main.py` | Entry point. Subcommands: `backtest`, `paper` (demo), `live --confirm` (real money, requires `--confirm` flag). |
| `agent_hq_server.py` | Dashboard web server (port 8080) |
| `dashboard_server.py` | Alternate dashboard (port 8080 too — don't run both) |
| `watchdog.py` | Container watchdog |
| `auto_trader.py` | Auto-trading logic |
| `trade_memory.db` | SQLite — table `trades` (source of truth for stats) |
| `learned_config.json` | Self-optimized config (written by LEARN agent) |
| `optimized_config.json` / `optimized_config_v2.json` | Manual optimization outputs |
| `docker-compose.yml` | Container definition |
| `live_trading.pid` / `live_trading_output.log` | Bot PID + stdout |
| `docs/` | Documentation |
| `src/` | Source modules (indicators, agents, strategies) |

## DB schema (table `trades`)

Key columns: `signal` (BUY/SELL), `entry_price`, `exit_price`, `pnl_usd`, `close_reason` (TP/SL), `regime` (TRENDING/RANGING/...), `confidence` (0–1), `r_achieved`, `entry_time`.

## docker-compose.yml essentials

```yaml
services:
  mt5:
    image: gmag11/metatrader5_vnc
    container_name: claw-trade-mt5
    restart: unless-stopped
    ports: ["3000:3000", "8001:8001"]
    volumes: [mt5_config:/config]
```