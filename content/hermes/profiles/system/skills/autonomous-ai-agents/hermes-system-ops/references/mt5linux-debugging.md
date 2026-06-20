# MT5 Linux Debugging Reference

Detailed debugging recipes for the mt5linux + Docker + Wine + RPyC stack.

## Architecture

```
Python bot (host) → mt5linux library → RPyC TCP (port 8001) →
  Docker container (claw-trade-mt5) → Wine →
    Python 3.9 32-bit + MetaTrader5 library →
      MT5 terminal64.exe (Wine) → Broker server
```

## Common Errors and Fixes

### Error: "Terminal: Authorization failed" (-6)

**Cause:** `mt5.initialize()` called without credentials, then `mt5.login()` separately.

**Fix:** Pass credentials to `initialize()` directly:
```python
# WRONG
mt5.initialize()
mt5.login(login, password, server)

# CORRECT
mt5.initialize(login=login_num, password=password, server=server)
```

### Error: "No IPC connection" (-10004)

**Cause:** MT5 terminal process is running but hasn't established IPC pipe yet,
or the terminal crashed and needs restart.

**Fix:**
1. Kill existing terminal: `docker exec claw-trade-mt5 bash -c "pkill -f terminal64"`
2. Wait 3 seconds
3. Restart: `docker exec -d -u abc -e DISPLAY=:1 -e WINEPREFIX=/config/.wine claw-trade-mt5 bash -c 'WINEDEBUG=-all wine "/config/.wine/drive_c/Program Files/MetaTrader 5/terminal64.exe" > /tmp/mt5.log 2>&1'`
4. Wait 30 seconds
5. Test connection

### Error: "Connection reset by peer" / EOFError

**Cause:** RPyC server not ready yet (still starting up in Wine).

**Fix:** Wait 30-60 seconds after container restart before connecting.
The RPyC server log shows "server started on 0.0.0.0:8001" when ready.

### Error: "tz_convert() takes exactly 2 positional arguments (1 given)"

**Cause:** pandas 3.0+ changed `tz_convert()` signature. mt5linux internally
calls `date.astimezone()` on tz-naive datetimes, which triggers the old
`tz_convert` code path.

**Fix:** Convert to UTC-aware datetime before passing to `copy_rates_range`:
```python
from datetime import timezone as dt_tz
start_dt = pd.to_datetime(start_date).to_pydatetime().replace(tzinfo=dt_tz.utc)
end_dt = pd.to_datetime(end_date).to_pydatetime().replace(tzinfo=dt_tz.utc)
rates = mt5.copy_rates_range(symbol, timeframe, start_dt, end_dt)
```

### Error: "Could not find any valid Gold symbol in MT5"

**Cause:** `symbol_select()` returns False even though the symbol exists.
Usually means MT5 terminal hasn't finished connecting to broker, or the
symbol isn't in Market Watch.

**Fix:**
1. Verify symbol exists: `mt5.symbols_get()` and filter for 'XAU'
2. Wait for MT5 terminal to fully connect (balance shows in account_info)
3. The connector already tries alternatives: `["XAUUSD", "GOLD", "XAUUSD.m", "XAUUSD.", "XAUUSD.i", "XAUUSD_", "XAUUSDc"]`

### Error: "OpenRouter HTTP 401: Missing Authentication header"

**Cause:** `.env` file has `OPENROUTER_API_KEY=***masked***` (literal asterisks)
or the process didn't load `.env`.

**Fix:**
1. Check `.env` has real key: `grep OPENROUTER_API_KEY /root/Claw_Trade/.env`
2. Verify `load_dotenv(override=True)` is called in config.py
3. Restart the bot process so it picks up the correct env

### Error: "Connection closed by peer" / "stream has been closed"

**Cause:** The RPyC connection dropped mid-operation. Often happens when
MT5 terminal crashes or restarts while the bot is connected.

**Fix:** Restart the bot process. The watchdog should handle this.

## Startup Sequence (after full container restart)

1. **0s:** `docker restart claw-trade-mt5`
2. **~5s:** Wine starts, s6 services launch
3. **~10-15s:** MT5 terminal64.exe launches in Wine
4. **~15-20s:** RPyC server (mt5linux) starts on port 8001
5. **~20-30s:** MT5 terminal auto-logins with broker
6. **~30-40s:** RPyC accepts connections, MT5 API ready
7. **~40s+:** Bot process can connect and start trading

Total: allow 40-60 seconds from container restart to bot connect.

## Key Files

| File | Purpose |
|------|---------|
| `/root/Claw_Trade/.env` | API keys, MT5 credentials, trading config |
| `/root/Claw_Trade/src/mt5_connector.py` | MT5 connection + order execution |
| `/root/Claw_Trade/src/data_handler.py` | Historical data fetching + indicators |
| `/root/Claw_Trade/src/agents.py` | Multi-agent AI analysis (Bull, Bear, CEO) |
| `/root/Claw_Trade/src/orchestrator.py` | Main trading loop coordinator |
| `/root/Claw_Trade/watchdog_v2.py` | Health check + auto-remediation script |
| `/root/Claw_Trade/live_state.json` | Trades today, last trade timestamp |
| `/root/Claw_Trade/live_positions.json` | Active positions |
| `/root/Claw_Trade/dashboard_server.py` | Retro pixel-art trading dashboard (port 8080) |
| `/tmp/claw_live*.log` | Bot stdout/stderr log (rotating) |

## Dashboard Access

- **Hermes Dashboard:** `http://<IP>:9119/?profile=system`
- **ClawTrade Dashboard:** `http://<IP>:8080` (requires `python3 dashboard_server.py` running)
- **MT5 VNC:** `http://<IP>:3000`

## Watchdog Cron Prompt (Trader Profile)

The Trader profile has a cron job "ClawTrade AI Watchdog" running every 15 minutes.
Its prompt instructs the agent to:

1. Run `python3 /root/Claw_Trade/watchdog_v2.py`
2. If exit 0 → silent ("OK")
3. If exit 1 → read error, fix based on error type, re-verify
4. Output one-line summary

Fix actions defined in the cron prompt:
- Container down → `docker restart claw-trade-mt5`
- MT5 terminal not running → start via `docker exec -d ... wine terminal64.exe`
- RPyC not listening → start mt5linux via `docker exec -d ... wine python.exe -m mt5linux`
- Bot not running → `cd /root/Claw_Trade && python3 main.py live ...`

Output files: `/root/.hermes/profiles/trader/cron/output/bd695ea2081d/*.md`