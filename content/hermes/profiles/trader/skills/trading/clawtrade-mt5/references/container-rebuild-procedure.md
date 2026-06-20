# Container Rebuild Procedure (Wine Prefix Corruption Recovery)

Use this when `wine: could not load kernel32.dll, status c0000135` appears.
This means the Wine prefix is permanently corrupted and must be recreated from scratch.

## Symptoms
- `wine cmd /c echo hello` → `c0000135`
- `wineboot --init` → creates prefix but kernel32.dll unloadable
- `wine python --version` → `Application could not be started`
- MT5 terminal64.exe won't launch

## Root Cause
Killing Wine system processes (wineserver, wineboot, winedevice) with `kill -9`
corrupts the prefix. The only fix is a full rebuild with a fresh volume.

## Full Rebuild Steps

### 1. Destroy container + volume
```bash
docker stop claw-trade-mt5
docker rm claw-trade-mt5
docker volume rm claw-trade-config 2>/dev/null  # remove old volume
```

### 2. Recreate with fresh volume
```bash
docker run -d \
  --name claw-trade-mt5 \
  -p 3000:3000 \
  -p 8001:8001 \
  -v claw-trade-config:/config \
  gmag11/metatrader5_vnc
```

### 3. Wait for container init (~25-30s)
```bash
sleep 30
```

### 4. Kill xmrig (always present in this image)
```bash
docker exec claw-trade-mt5 pkill -9 -f xmrig
docker exec claw-trade-mt5 pkill -9 -f xmr_linux
docker exec claw-trade-mt5 rm -rf /tmp/xmrig /tmp/xmr_linux_amd64
```

### 5. Verify Wine works
```bash
docker exec -u abc -e WINEPREFIX=/config/.wine -e WINEDEBUG=-all \
  claw-trade-mt5 bash -c 'wine cmd /c echo hello'
# Should print "hello"
```

### 6. Run start.sh to install MT5 + Python + mt5linux
```bash
docker exec -d -u abc -e DISPLAY=:1 -e WINEPREFIX=/config/.wine -e WINEDEBUG=-all \
  claw-trade-mt5 bash -c 'cd /Metatrader && bash start.sh > /tmp/mt5_install.log 2>&1'
```
This downloads and installs:
- Wine Mono (~81MB)
- MetaTrader 5 terminal (~22MB)
- Python 3.9 32-bit (~26MB)
- MetaTrader5 Python library (5.0.36)
- mt5linux + rpyc + numpy
- python-dateutil

**Takes ~5-10 minutes.** Monitor with:
```bash
docker exec claw-trade-mt5 bash -c 'tail -20 /tmp/mt5_install.log'
```

### 7. Apply numpy fix to Wine-side metatrader5.py
```bash
docker exec claw-trade-mt5 bash -c 'sed -i "20 a\\        self.__conn.execute(\"import numpy as np\")" \
  "/config/.wine/drive_c/Program Files (x86)/Python39-32/Lib/site-packages/mt5linux/metatrader5.py"'
```

### 8. Open iptables for VNC (if blocked)
```bash
iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
```

### 9. User must login MT5 via VNC
- Open `http://SERVER_IP:3000` in browser
- MT5 terminal should be visible (start.sh launches it)
- Login with credentials (login 106123714, server FBSTradestone-Demo)
- **Fresh install = no saved session. User MUST login manually.**

### 10. Start mt5linux RPyC server
```bash
docker exec -d -u abc -e WINEPREFIX=/config/.wine -e WINEDEBUG=-all \
  claw-trade-mt5 bash -c 'python3 -m mt5linux --host 0.0.0.0 --port 8001 \
    -w wine "C:\Program Files (x86)\Python39-32\python.exe" > /tmp/mt5linux.log 2>&1'
sleep 15
# Verify
docker exec claw-trade-mt5 bash -c 'ss -tlnp | grep 8001'
```

### 11. Verify MT5 connection
```python
import mt5linux, os
from dotenv import load_dotenv
load_dotenv('/root/Claw_Trade/.env')
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
ok = mt5.initialize(login=106123714, password='...', server='FBSTradestone-Demo')
print('Connected:', ok)
print('Account:', mt5.account_info())
mt5.shutdown()
```

### 12. Update .env credentials
The `.env` file may have placeholder values (`***`) after git operations.
Ensure these are real:
- `MT5_PASSWORD` — actual MT5 password
- `OPENROUTER_API_KEY` — real OpenRouter API key (free tier works)

### 13. Start bot
```bash
cd /root/Claw_Trade && python3 main.py live --confirm --symbol XAUUSDc --interval 5 &
```

## Key Notes

- **Wine 10.0** in this image uses `wine` (not `wine64`) for 64-bit prefix
- **rpyc version**: Both host and Wine Python must match (5.2.3). The start.sh
  installs rpyc 5.2.3 inside Wine Python. Host must also have rpyc 5.2.3.
- **MT5 terminal title** shows login status: `106123714 - FBSTradestone-Demo: Demo Account`
  means logged in. Check with `wmctrl -l` or `xwininfo -tree -root`.
- **xdotool may not find MT5 windows** even when they exist. Use `xwininfo -tree -root`
  or `wmctrl -l` instead.
- **Screenshot via ffmpeg**: `DISPLAY=:1 ffmpeg -y -f x11grab -video_size WxH -i :1 -frames 1 /tmp/screen.png`
  (check actual screen size first — it may differ from 1024x768)