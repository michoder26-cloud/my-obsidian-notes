# Syncthing + Obsidian Vault Sync (VPS ↔ Windows)

## Overview
Set up automatic bidirectional sync between VPS and Windows desktop so the user can edit Obsidian notes locally while the agent can read/write them on the VPS.

## Setup Steps

### 1. Install Syncthing on VPS
```bash
curl -s https://syncthing.net/release-key.gpg | gpg --dearmor | tee /usr/share/keyrings/syncthing-archive-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable" > /etc/apt/sources.list.d/syncthing.list
apt-get update -qq && apt-get install -y -qq syncthing
```

### 2. Create vault folder on VPS
```bash
mkdir -p /root/obsidian-vault/{trading,system,cron-output,journal,security}
# Copy interesting .md files from Hermes into vault
cp /root/.hermes/cron/output/*/*.md /root/obsidian-vault/cron-output/ 2>/dev/null
cp /root/.hermes/skills/trading/*/SKILL.md /root/obsidian-vault/trading/ 2>/dev/null
cp /root/.hermes/skills/security/security-monitoring/SKILL.md /root/obsidian-vault/security/ 2>/dev/null
```

### 3. Generate config + get VPS Device ID
```bash
syncthing generate
# Output: Device ID: XXXXXXX-XXXXXXX-...
```

### 4. Start Syncthing on VPS (background, no browser)
```bash
# Use terminal(background=true) — NOT nohup/&
syncthing --no-browser --gui-address=127.0.0.1:8384
```

### 5. Edit config XML to add Windows device
Config path: `/root/.local/state/syncthing/config.xml`

- Replace default folder path with `/root/obsidian-vault`
- Add Windows device ID inside `<folder>` and as a top-level `<device>`
- Create `.stfolder` marker: `mkdir -p /root/obsidian-vault/.stfolder`
- Restart syncthing after editing config

### 6. Windows side setup
1. Download Syncthing from syncthing.net
2. Open GUI at `http://127.0.0.1:8384`
3. Add Remote Device → paste VPS Device ID
4. Add Folder → select local Obsidian vault path
5. Share with VPS device
6. **IMPORTANT: Do NOT mark VPS as "Untrusted"** (see pitfall below)

## Pitfalls

### ⚠️ "remote expects to exchange encrypted data, but is configured for plain data"
**Symptom**: Devices connect (green briefly) then disconnect immediately. Log shows:
```
Lost primary connection to DEVICE: handling cluster-config: remote expects to exchange encrypted data, but is configured for plain data
```
**Cause**: Windows side marked VPS device as "Untrusted", which forces encrypted folder. VPS is plain.
**Fix**: On Windows Syncthing GUI → Devices → VPS → **UNCHECK "Untrusted"** → Save. Or delete device and re-add without untrusted flag.

### ⚠️ Can't use `&` or `nohup` to start Syncthing
Hermes terminal tool blocks `&` backgrounding. Use `terminal(background=true)` instead.

### ⚠️ Syncthing config XML editing
The `syncthing cli` commands have inconsistent syntax across versions. Editing config.xml directly is more reliable:
- Stop syncthing first: `pkill syncthing`
- Edit `/root/.local/state/syncthing/config.xml`
- Restart with `terminal(background=true)`