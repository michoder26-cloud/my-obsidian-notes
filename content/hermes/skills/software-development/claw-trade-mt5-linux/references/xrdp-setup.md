# xrdp Remote Desktop — ⚠️ DEPRECATED (Removed June 2026)

> **xrdp was removed** in favor of **NoMachine (port 4000)** due to persistent lag on AWS (software render + latency).
> 
> This file is kept for historical reference only. Do NOT install xrdp again — it causes window manager crashes
> and the compositor fix breaks reconnect. Use NoMachine instead (see parent SKILL.md for install steps).
> All `analyst` user operations (terminal, code, git) work fine via SSH.
> 
> To block port 3389 if xrdp was ever re-installed: `iptables -A INPUT -p tcp --dport 3389 -j DROP`

## Architecture: Two Desktop Systems

This server runs TWO separate remote desktop systems serving different purposes:

### System A: xrdp (port 3389) — Host Desktop
- **User**: `analyst`
- **X Server**: Xorg (real X server on the host)
- **Access**: Windows Remote Desktop Connection (mstsc.exe) → `13.140.183.183:3389`
- **Purpose**: Full desktop for coding, terminal, git, running Python bot
- **Mounts**: Can access `/root/Claw_Trade/` and all host files

### System B: KasmVNC (port 3000) — Docker Desktop
- **User**: `abc` (inside Docker container)
- **X Server**: Xvnc (virtual X server, headless)
- **Access**: Browser → `http://13.140.183.183:3000`
- **Purpose**: Only for opening MT5 Terminal to login/manage positions
- **Cannot**: Access host files or run Python bot

**They are completely separate** — the only bridge is mt5linux RPyC on port 8001.

## Performance Optimization

### Problem: xrdp is laggy/stuttering
**Causes**: AWS has no GPU (software render) + latency from Thailand (~30-50ms) + Xfce compositor effects.

### Fix 1: Lower color depth
```bash
sudo sed -i 's/max_bpp=16/max_bpp=15/' /etc/xrdp/xrdp.ini
sudo systemctl restart xrdp
```
Cuts bandwidth nearly in half. Visual difference is barely noticeable for terminal/code work.

### Fix 2: Disable Xfce compositor
```bash
# Prerequisite — dbus-x11 must be installed or xfconf-query will fail:
sudo apt install dbus-x11 -y

# Find which display the analyst session is on
ps aux | grep "Xorg.*analyst" | grep -v grep | head -1
# Example output shows :11 — adjust display number as needed
sudo -u analyst DISPLAY=:11 xfconf-query -c xfwm4 -p /general/use_compositing -s false
```
Removes window shadows, transparency, and fade effects.

### Fix 3: Alternative — NoMachine
If xrdp is still too slow, install NoMachine (NX protocol — better over high-latency):
```bash
wget https://download.nomachine.com/download/8.16/Linux/nomachine_8.16.1_1_amd64.deb
sudo dpkg -i nomachine_8.16.1_1_amd64.deb
sudo systemctl stop xrdp  # free up port 3389 if needed
```

## Troubleshooting

### Symptom: RDP connects then immediately disconnects
**Log check:**
```bash
tail -30 /var/log/xrdp-sesman.log
tail -30 /var/log/xrdp.log
```

**Common error:**
```
[WARN] Window manager (pid XXX, display YY) exited quickly (1 secs).
This could indicate a window manager config problem
```

**Fixes (in order of likelihood):**

**Fix A — Restart xrdp (fastest):**
```bash
pkill -9 xrdp xrdp-sesman xrdp-chansrv 2>/dev/null
sleep 1
rm -f /var/run/xrdp/xrdp.pid /tmp/.X*-lock /tmp/.X11-unix/X* /run/xrdp/sockdir/xrdp_*
/usr/sbin/xrdp
```

**Fix B — Revert compositor change:**
```bash
sudo -u analyst DISPLAY=:11 xfconf-query -c xfwm4 -p /general/use_compositing -s true
# Then restart xrdp
```

**Fix C — Clean all sessions (nuclear):**
```bash
pkill -9 xrdp xrdp-sesman xrdp-chansrv Xorg 2>/dev/null
sleep 2
rm -f /tmp/.X*-lock /tmp/.X11-unix/X* /run/xrdp/sockdir/xrdp_* /var/run/xrdp/xrdp.pid
/usr/sbin/xrdp
```

### Symptom: "Cannot read private key file /etc/xrdp/key.pem: Permission denied"
Non-critical warning. RDP falls back to non-TLS mode. The session works, just not encrypted. To fix permanently:
```bash
sudo chmod 644 /etc/xrdp/key.pem /etc/xrdp/cert.pem
```

### Symptom: xrdp-sesman not running
Check: `ps aux | grep xrdp-sesman`
If missing, xrdp can't create sessions. Start it:
```bash
/usr/sbin/xrdp-sesman
# Or just restart xrdp which should auto-spawn it:
pkill xrdp; /usr/sbin/xrdp
```

## Port Layout

| Port | Service | Who uses it |
|------|---------|-------------|
| 22 | SSH | Terminal access |
| 3389 | xrdp (RDP) | analyst — Remote Desktop |
| 3000 | KasmVNC (Web) | Docker container desktop |
| 8001 | mt5linux (RPyC) | Python bot ↔ MT5 bridge |