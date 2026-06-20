---
name: gemini-cli-headless
description: "Use Gemini CLI on headless servers - authentication, desktop access, and subscription integration."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [gemini, cli, headless, authentication, desktop]
---

# Gemini CLI on Headless VPS

Guide for using Google Gemini CLI (`$19.99/month` subscription) on headless servers where OAuth browser login is impossible.

## Key Insight

**Gemini CLI is NOT an MCP server.** It cannot be added to `mcp_servers` in `config.yaml`. It uses:
- `--acp` for Agent Communication Protocol (IDE integration)
- OAuth browser flow for authentication (fails headless)
- `GEMINI_API_KEY` env var for headless authentication (works with paid account)

## Authentication Methods

### Method 1: API Key (Recommended for Headless)

Even paid Google One AI Premium subscribers use API Key for headless:

```bash
# Get API Key: https://aistudio.google.com/apikey (login with paid account)
# The API key inherits your $19.99/month subscription benefits

# Valid format: AIzaSy... (39-40 characters)
echo 'GEMINI_API_KEY="AIzaSy..."' >> ~/.hermes/.env
export GEMINI_API_KEY="AIzaSy..."
gemini -p "hello" --output-format json
```

**Important:** In gemini-cli 0.46.0, setting `"selectedType": "api-key"` in settings.json causes "Invalid auth method selected". Remove this setting or use OAuth flow.

### Method 2: Desktop Access (Interactive) - Rarely Needed

**Important:** Google OAuth tokens (format: `AQ.A...`) are NOT valid Gemini API Keys.
- OAuth tokens: `AQ.A...` format (from browser login) - ใช้ใน browser เท่านั้น
- API Keys: `AIzaSy...` format (39-40 chars) - ใช้ใน CLI

For OAuth login (rarely needed), access desktop via:

| Option | Port | Notes |
|--------|------|-------|\n| Docker GUI container | 5900 | MT5 container (claw-trade-mt5) has VNC on 5900 - **แต่ไม่มี browser ใช้ได้** |\n| NoMachine | 4000 | Full desktop, already configured |\n| VNC Desktop | 5901 | TigerVNC with XFCE, use VNC client |

## Desktop Setup

### VNC Desktop (TigerVNC)
```bash
# Already running on port 5901
# Password: same as SSH password

# Create VNC password
echo '***' | vncpasswd -f > /root/.vnc/passwd
vncserver :1 -geometry 1280x720 -depth 24 -passwd /root/.vnc/passwd

# External access: VNC client to 13.140.183.183:5901
```

### NoMachine (if reinstallable)
```bash
# Download from nomachine.com directly (apt package is incomplete)
wget https://download.nomachine.com/.../nomachine_*.deb
dpkg -i nomachine_*.deb
/etc/NX/nxserver --start
```

## Docker Desktop Alternatives

### Pitfall: jlesage/firefox has NO built-in terminal
Image runs Firefox only, no shell access. You cannot run `gemini auth login` inside it.

### Pitfall: consol/ubuntu-xfce-vnc uses Ubuntu 16.04 (xenial)
libc too old for Node.js 20+. Node 12 won't run `@google/gemini-cli`.

### Pitfall: snap Firefox doesn't work in VNC containers
snapd socket missing. Use `apt install firefox` instead.

### Pitfall: dorowu/ubuntu-desktop-lxde-vnc port mapping
Map explicitly: `-p 6080:5800 -p 5900:5900` (noVNC on 6080, VNC on 5900).

## Package Installation Pitfills

### dpkg lock conflicts
```bash
# Fix: dpkg --configure -a
apt-get install -y firefox nodejs npm
```

### NoMachine apt package incomplete
Use `.deb` from nomachine.com directly:
```bash
wget https://download.nomachine.com/.../nomachine_*.deb
dpkg -i nomachine_*.deb
```

### NoMachine + snap Firefox fails
Snap confinement blocks Firefox in NoMachine desktop:
- Error: `is not a snap cgroup`
- Error: `cannot open display: :10.0`
- **Fix:** `apt install firefox` after removing snap, or use VNC Desktop

## Subscription Notes

- Google One AI Premium ($19.99/month) = Gemini 2.5 Pro access
- API Key created with paid account → uses subscription quota
- No separate credit purchase needed

## Delegation Pattern

Use Gemini CLI as sub-agent via `delegate_task`:
```python
delegate_task(goal="Analyze this trading strategy using Gemini CLI")
```