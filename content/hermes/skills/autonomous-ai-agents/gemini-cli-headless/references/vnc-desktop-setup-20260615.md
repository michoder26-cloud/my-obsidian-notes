# VNC Desktop Setup Log - 2026-06-15

## Session Context
User's VPS (13.140.183.183) needed Gemini CLI auth via browser on headless server.

## Failed Attempts

### Attempt 1: jlesage/firefox (port 5800)
- **Result:** No terminal access in noVNC interface
- **Lesson:** Image designed for single-app (Firefox) only

### Attempt 2: consol/ubuntu-xfce-vnc (port 6081)
- **Result:** Ubuntu 16.04 (xenial) - libc too old for Node 20+
- **Error:** `nodejs : Depends: libc6 (>= 2.28) but 2.23 is installed`
- **Lesson:** Prefer Ubuntu 22.04+ images for modern Node.js

### Attempt 3: oommen81/xfce (port 6080/6081)
- **Result:** Has XFCE + terminal but:
  - noVNC returns "Connection reset" (networking issue)
  - Node.js 12.22.9 (too old for gemini-cli)
  - Firefox present but gemini-cli fails with SyntaxError

### Attempt 4: NoMachine (port 4000)
- **Result:** Full Ubuntu desktop available
- **Problem:** snap Firefox blocked by confinement
- **Error:** `/snap/bin/firefox: "not a snap cgroup"` + display mismatch `:10.0`

## Working Configuration

### Recommended Docker Desktop Setup
```bash
docker run -d \
  --name=gemini-desktop \
  -p 6080:6080 \
  -p 5900:5900 \
  -e VNC_PASSWORD=*** \
  -v ~/.gemini:/root/.gemini \
  --shm-size=512m \
  oommen81/xfce:latest

# Then install modern Node.js
docker exec gemini-desktop curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
docker exec gemini-desktop apt-get install -y nodejs
docker exec gemini-desktop npm install -g @google/gemini-cli
```

## Root Cause Analysis
1. **Old Ubuntu versions** → libc compatibility breaks Node.js 20+
2. **snap confinement** → prevents Firefox launch in containerized desktop
3. **jlesage images** → no terminal for authentication flow
4. **NoMachine desktop** → user session mismatch with snap apps

## Quick Resolution
Use API Key instead: `GEMINI_API_KEY` env var with paid Google account subscription.