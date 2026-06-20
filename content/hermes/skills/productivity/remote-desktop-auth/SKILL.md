---
name: Remote Desktop Auth
tags:
  - remote-desktop
  - vnc
  - nomachine
  - x2go
  - ssh-x11
description: Set up GUI desktop access for browser-based authentication on headless VPS
---

# Remote Desktop for Headless VPS Authentication

User preference: Uses NoMachine or VNC to access Ubuntu desktop for OAuth browser authentication, not API key workaround.

## Prerequisites

- Ubuntu/Debian VPS
- SSH access with password login enabled
- Non-root user 'analyst' (or any user with desktop session)

## Quick Setup Options

### Option 1: TigerVNC (Built into Ubuntu)

```bash
# Install XFCE desktop
apt-get install -y xfce4 xfce4-goodies tightvncserver

# Set VNC password
mkdir -p /root/.vnc
echo 'your-password' | vncpasswd -f > /root/.vnc/passwd
chmod 600 /root/.vnc/passwd

# Start VNC server
vncserver :1 -geometry 1280x720 -depth 24 -passwd /root/.vnc/passwd -localhost no
```

Connect: `vnc-client your-vps-ip:5901`

### Option 2: X2Go (SSH-based, more reliable)

```bash
# Install X2Go server
apt-get install -y x2goserver x2goserver-xsession

# Note: X2Go uses SSH port 22, no additional ports needed
```

Connect: X2Go Client → Host: `vps-ip`, Port: `22`, Session type: `XFCE`

### Option 3: NoMachine (Fastest performance)

```bash
# Download from: https://www.nomachine.com/download
# Install .deb package
dpkg -i nomachine_*.deb
/usr/NX/scripts/setup/nxserver --install

# Status: systemctl status nxserver
```

Connect: NoMachine client → `vps-ip:4000`

### Option 4: Docker Desktop (Isolated)

```bash
docker run -d \
  --name=gemini-desktop \
  -p 6080:5800 -p 5900:5900 \
  -e VNC_PASSWORD=*** \
  -v /root/.gemini:/config/.gemini \
  --shm-size=512m \
  jlesage/firefox:latest
```

Connect: `http://vps-ip:6080/`

## Pitfalls

1. **Snap Firefox confinement**: Firefox from snap won't work in NoMachine/X2Go. Install from apt: `apt-get install -y firefox`
2. **Node.js version**: Gemini CLI requires Node.js ≥ 20. Check with `node --version`
3. **VNC password file**: Must be chmod 600, password in plain text won't work - use `vncpasswd -f`
4. **Port conflicts**: Kill old containers before starting new ones: `docker rm -f old-container`
5. **Display mismatch**: SSH sessions and VNC may use different display numbers (`:0` vs `:1`)
6. **NoMachine .deb download redirect**: Official download URLs redirect to HTML pages, use package manager or known working mirrors

## Verification

After connecting desktop:
```bash
# Check desktop
echo $XDG_SESSION_TYPE  # Should be 'x11'

# Check browser
firefox --version  # Or chromium-browser

# Check terminal
xfce4-terminal --version  # Or gnome-terminal

# Check gemini CLI
gemini --version
node --version  # Must be ≥ 20
```

## User Preferences

- Prefers NoMachine for speed and stability
- Needs working browser + terminal together
- Uses Thai language communication
- Prefers direct concise answers