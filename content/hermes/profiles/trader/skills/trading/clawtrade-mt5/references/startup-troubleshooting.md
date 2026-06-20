# ClawTrade Startup Troubleshooting

## Diagnostic Flow (top-down — check each, fix before moving up)

### 1. Bot not producing trades
Check `/tmp/claw_live5.log` (or current bot log). Look for:
- `OpenRouter HTTP 401` → API key is placeholder `***` in `.env`. Replace with real key.
- `'dict' object has no attribute 'balance'` → orchestrator.py not patched. See Fix 2 in SKILL.md.
- `name 'np' is not defined` → mt5linux not patched. See Fix 1 in SKILL.md.
- `CEO Decision: NO_TRADE | Confidence: 0.00` → agents failed to analyze (check OpenRouter errors above).

### 2. Bot process not running
```bash
ps aux | grep "main.py" | grep "live" | grep -v grep
```
If empty: start with `cd /root/Claw_Trade && python3 main.py live --confirm --symbol XAUUSDc --interval 5 &`

### 3. MT5 not logged in (terminal_info returns None)
MT5 terminal must be running in Wine AND logged in. Two scenarios:
- **Terminal not running**: Launch via Wine (see SKILL.md step 3). Wait 15-20s.
- **Terminal running but not logged in**: Need to log in via VNC web UI (http://VPS-IP:3000)
  or call `mt5.initialize(login=..., password=..., server=...)` with credentials from `.env`.

Verify:
```python
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
mt5.initialize(login=LOGIN, password=PASSWORD, server=SERVER)
print(mt5.terminal_info())  # Should NOT be None
print(mt5.account_info())    # Should show balance, login, etc.
```

### 4. RPyC port 8001 not listening
```bash
docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep 8001'
```
If empty: start mt5linux manually (see SKILL.md step 4). The s6 service supervisor may have
crashed if MT5 terminal wasn't ready when it tried to start.

### 5. MT5 terminal not in Wine
```bash
docker exec -u abc claw-trade-mt5 bash -c 'WINEPREFIX=/config/.wine wine tasklist 2>&1 | grep terminal64'
```
If empty: launch terminal64.exe (see SKILL.md step 3).

### 6. Container not running
```bash
docker ps --filter name=claw-trade-mt5
```
If empty: `docker start claw-trade-mt5` or `docker compose -f /root/Claw_Trade/docker-compose.yml up -d`

### 7. xmrig consuming resources
```bash
docker exec claw-trade-mt5 bash -c 'ps aux --sort=-%cpu | head -5'
```
If xmrig/xmr_linux at top: kill it (see SKILL.md pitfalls). The watchdog also handles this.

## Common Error → Fix Mapping

| Error | Layer | Fix |
|-------|-------|-----|
| `OpenRouter HTTP 401` | Bot/Agents | Real API key in `.env` |
| `'dict' has no attribute 'balance'` | Orchestrator | Patch orchestrator.py line ~676 |
| `name 'np' is not defined` | mt5linux/RPyC | Patch mt5linux metatrader5.py |
| `terminal_info() = None` | MT5 terminal | Start terminal + login |
| `account_info() = None` | MT5 login | Call mt5.initialize with credentials |
| `wine: could not load kernel32.dll, status c0000135` | Wine prefix corrupted | See `references/container-rebuild-procedure.md` — full container rebuild required |
| `ValueError: not enough values to unpack (expected 3, got 0)` | RPyC version mismatch | Ensure host and Wine Python rpyc versions match (both should be 5.2.3) |
| `Application could not be started, or no application associated` | Fresh MT5 install | Launch terminal64.exe separately; requires VNC login at http://VPS-IP:3000 |
| `Connection reset by peer` on RPyC connect | RPyC server down/crashed | Restart mt5linux server in container (see SKILL.md step 4) |
| `wine: not owned by you` | Wine perms | Run as user `abc`: `docker exec -u abc -e WINEPREFIX=/config/.wine ...` |
| `Cannot GET /api/overview` on :8001 | Wrong server | Port 8001 is RPyC, not HTTP. Dashboard is :8080 (agent_hq_server.py) |
| `tz_convert() takes exactly 2 positional arguments` | pandas version mismatch | Non-critical, bot falls back to Yahoo Finance |

## After Container Restart (full recovery sequence)

1. Wait 30s for container to stabilize
2. Kill xmrig
3. chown wine prefix: `docker exec claw-trade-mt5 chown -R abc:abc /config/.wine`
4. Launch MT5 terminal in Wine
5. Wait 15-20s for MT5 to initialize
6. Start mt5linux RPyC server (if s6 didn't auto-start it)
7. Verify connection with mt5.initialize + terminal_info
8. Start bot: `python3 main.py live --confirm --symbol XAUUSDc --interval 5`
9. Run watchdog to confirm all layers: `python3 watchdog_v2.py`