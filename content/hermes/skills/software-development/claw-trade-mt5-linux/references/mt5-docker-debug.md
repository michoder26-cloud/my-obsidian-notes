# MT5 Docker Debug Reference

## Quick Health Check (run in order)

```bash
# 1. Container running?
docker ps | grep claw-trade-mt5

# 2. Ports listening?
ss -tlnp | grep -E "8001|3000"

# 3. mt5linux process inside container?
docker exec claw-trade-mt5 ps aux | grep python.exe

# 4. Bot process running?
ps aux | grep "main.py" | grep -v grep

# 5. Latest errors?
tail -50 /root/Claw_Trade/live_trading_output.log
```

## Common Error Patterns

### "Symbol 'XAUUSDc' not found"
- MT5 not logged into broker → VNC in and login
- mt5linux not started → start manually (see skill SKILL.md)
- MT5 restart loop → `docker restart claw-trade-mt5`
- **Wine permission denied** → mt5linux silently failed (see "Wine Permission Denied" section above). Fix the s6 run script to use `su - abc -c` and restart.

### ConnectionRefusedError on port 8001
- Port is listened by docker-proxy but mt5linux process died
- Fix: `docker exec -d claw-trade-mt5 bash -c 'export WINEPREFIX=/config/.wine && wine "C:\\\\Program Files (x86)\\\\Python39-32\\\\python.exe" -m mt5linux --host 0.0.0.0 --port 8001 &'`

### 🔴 Wine Permission Denied — mt5linux Silently Fails
**Symptoms:** s6-supervise `mt5linux` service shows as running, but `docker exec claw-trade-mt5 ps aux | grep python.exe` shows NO process. Bot cannot connect (ConnectionRefusedError or EOFError on RPyC). Log may show nothing.

**Root cause:** `/config/.wine` is owned by user `abc`, but s6 runs the service script as `root`. Wine refuses to start with error: `wine: '/config/.wine' is not owned by you`.

**Diagnosis:**
```bash
# Check ownership
docker exec claw-trade-mt5 stat -c "%U:%G" /config/.wine
# If owner is NOT the user running wine → problem confirmed

# Check if mt5linux process exists
docker exec claw-trade-mt5 ps aux | grep python.exe
# Empty (except s6-supervise) → wine never started
```

**Fix:**
```bash
# Run mt5linux as the correct user
docker exec -d claw-trade-mt5 bash -c "su - abc -c 'export WINEPREFIX=/config/.wine && wine \"C:\\\\Program Files (x86)\\\\Python39-32\\\\python.exe\" -m mt5linux --host 0.0.0.0 --port 8001'"

# OR permanently fix the s6 run script
docker exec claw-trade-mt5 bash -c 'cat > /run/service/mt5linux/run << '"'"'EOF'"'"'
#!/usr/bin/with-contenv bash
export WINEPREFIX=/config/.wine
exec su - abc -c "export WINEPREFIX=/config/.wine && wine \"C:\\\\Program Files (x86)\\\\Python39-32\\\\python.exe\" -m mt5linux --host 0.0.0.0 --port 8001"
EOF
chmod +x /run/service/mt5linux/run'

# Then restart the service
docker exec claw-trade-mt5 s6-svc -d /run/service/mt5linux
docker exec claw-trade-mt5 s6-svc -u /run/service/mt5linux

# Verify (wait ~10s for MT5 to fully load)
sleep 10
docker exec claw-trade-mt5 ps aux | grep python.exe
# Should now show python.exe -m mt5linux process
```

**Note:** After fixing, existing duplicate mt5linux processes may need cleanup: `docker exec claw-trade-mt5 kill <old_pid>`.

### Too many start.sh processes (>5)
- MT5 is in a restart loop inside container
- Fix: `docker restart claw-trade-mt5` (cleaner than killing individual processes)

## Log Locations
| File | Purpose |
|------|---------|
| `/root/Claw_Trade/live_trading_output.log` | Bot activity, trade signals, errors |
| `/root/Claw_Trade/live_trading_watchdog.log` | Watchdog restart history |
| `/root/Claw_Trade/trade_history_log.json` | Trade records |

## Recovery Time Estimates
| Issue | Time to fix |
|-------|-------------|
| mt5linux restart | ~10s |
| Wine permission fix + mt5linux restart | ~30s |
| Docker container restart + MT5 load | ~60s |
| Manual MT5 broker login via VNC | ~2min |
