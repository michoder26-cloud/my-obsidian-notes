# 🩹 Troubleshooting — real problems + fixes

This document collects problems actually hit in real use, with targeted fixes, ordered from "most common" down.

---

## 📑 Problem index

1. [Characters don't show on Telegram but do on CLI](#1-characters-dont-show-on-telegram-but-do-on-cli)
2. [Old sessions (started before install) don't send events](#2-old-sessions-started-before-install-dont-send-events)
3. [Build fails on Windows (prettier "No files matching")](#3-build-fails-on-windows-prettier-no-files-matching)
4. ["remote port forwarding failed" or the tunnel keeps dying](#4-remote-port-forwarding-failed-or-the-tunnel-keeps-dying)
5. [Token mismatch after restarting the office](#5-token-mismatch-after-restarting-the-office)
6. [Check whether events are arriving](#6-check-whether-events-are-arriving)
7. [Characters "disappear"](#7-characters-disappear)
8. [Want to see characters move clearly](#8-want-to-see-characters-move-clearly)

---

## 1) Characters don't show on Telegram but do on CLI

### Symptom

- Running `hermes` in the CLI → the character moves in the office ✅
- Chatting through the Telegram bot → Hermes replies, but the character **does not move** ❌

### Cause

You are using the **`pixel_observer` plugin** (Python) instead of the bridge. The problem is that the Hermes gateway/Telegram **only fires shell hooks**, not plugin hooks, while the CLI does load the plugin. So it looks like "only half works."

### Fix

**Switch to the bridge (shell hooks), not the plugin:**

1. Install the shell hooks in `~/.hermes/config.yaml` (see `config/hermes-hooks.yaml.snippet` or the Install section of the README)
2. Make sure the line `hooks_auto_accept: true` is present
3. Restart the gateway once (see item 2)

> 📖 Full detail at: [`PLUGIN-VS-BRIDGE.md`](PLUGIN-VS-BRIDGE.md)

---

## 2) Old sessions (started before install) don't send events

### Symptom

After installing the bridge/hooks, sessions that were **already running before** don't move in the office, but new sessions started after install do work.

### Cause

Hermes loads `~/.hermes/config.yaml` (including hooks) **only when a session starts**. A session that was already running still holds the old config in memory, so it does not know about the new hooks.

### Fix

Restart the gateway/profile **once** so it loads the new hooks:

```bash
# if you use a watchdog-style gateway
sudo systemctl restart hermes-gateway-watchdog

# or restart the profile your way (e.g. restart the service running that profile)
```

After this, **every new session** sends events. (Old stuck sessions need to finish first, or wait for the restart.)

> 💡 No rush — one restart is enough. Do not loop-restart, because in-flight work can be lost.

---

## 3) Build fails on Windows (prettier "No files matching")

### Symptom

When running `npm run build` on Windows you get:

```
[error] No files matching the pattern "..."
```

coming from `prettier`.

### Cause

The quoting/glob expansion of prettier behaves differently on a Windows shell (cmd/PowerShell) compared to bash, so patterns written for Unix can't find the files.

### Fix

Read the detailed fix at:

```
fixes/generate-messages.windows-quote-fix.md
```

In short, fix the quoting in the relevant `package.json` script (e.g. use double-quotes or adjust the glob) to suit Windows.

> 💡 **Tip:** if possible, build directly on the Linux/VPS machine and then deploy — it avoids a lot of shell headaches.

---

## 4) "remote port forwarding failed" or the tunnel keeps dying

### Symptom

You use an SSH reverse tunnel (`ssh -R ...`) to pull the office from the VPS to your laptop and hit:

```
Warning: remote port forwarding failed for listen port 3100
```

or the tunnel dies after a while and characters stop moving.

### Cause

This project is **designed to run the office on the same machine as Hermes (the VPS)**, not to use a tunnel at all. A reverse tunnel (`-R`) often clashes with a port already held on the VPS and is not stable.

### Fix

**Option 1 (recommended): don't use a tunnel at all** — run the office on the same VPS as Hermes and view it directly at `http://<YOUR_VPS_IP>:3100` (or bind `0.0.0.0`).

**Option 2 (if you must view from another machine): use an SSH FORWARD tunnel, not a reverse one**

```bash
ssh -L 3100:127.0.0.1:3100 <user>@<YOUR_VPS_IP>
```

Then open `http://127.0.0.1:3100` on your laptop. It is a forward tunnel (one-way, from laptop into the VPS), which is more stable and does not have the "port forwarding failed" problem.

> ⚠️ Never use `ssh -R` with this project — it will die on its own.

---

## 5) Token mismatch after restarting the office

### Symptom

You restart `pixel-office` and worry that the token in `server.json` changed → the bridge's POSTs will be rejected.

### Fix

**You do not need to do anything.** ✅

The bridge (`pixel_agents_bridge.py`) **re-reads `~/.pixel-agents/server.json` every time** it is called, so it sees the office's new token automatically. No Hermes restart, no config edit.

> 💡 This is exactly why discovery uses a file instead of a constant — it keeps the office and bridge from having to sync with each other.

---

## 6) Check whether events are arriving

When characters don't move, trace from the "source" downward:

### Step A — Look at the bridge (did Hermes fire the hook?)

```bash
tail -f ~/.hermes/pixel_agents_bridge.log
```

If event lines appear → Hermes fired the hook correctly; the problem is past the bridge. If it is empty → Hermes is not firing (go back to items 1 and 2).

### Step B — Look at the office (did the bridge's POST arrive?)

```bash
tail -f ~/pixel-office.log
```

Look for lines containing `Hermes:`, e.g.:

```
Hermes: on_session_start  profile=telegram
Hermes: pre_tool_call     profile=telegram tool=Bash
```

If they are present → the office received the event; the problem is probably the browser/WebSocket (try refreshing the page). If empty → the bridge's POST is not arriving (check the token in `server.json`, the port, and whether the office is running).

### Step C — Check the office is running

```bash
sudo systemctl status pixel-office
curl http://127.0.0.1:3100/api/health
```

---

## 7) Characters "disappear"

### Symptom

You open the office and don't see the expected character, or it shows briefly then vanishes.

### Cause

Characters **appear when active and go sit idle when idle** — when idle they have not "vanished," they are just sitting still (possibly off-screen edge, or small enough to overlook). This is normal behavior, not a bug.

### Fix

1. **Hard refresh the page** — `Ctrl+F5` to clear the old WebSocket
2. **Send a message to Hermes while the page is open** — once there is activity, the character will move into view
3. If you still don't see it → trace the logs per item 6

---

## 8) Want to see characters move clearly

### Symptom

You open the page and the characters "barely move," leaving you unsure whether it is really working.

### Fix

Send a task that makes Hermes **call tools several times**, e.g.:

- "Read all the files in folder X and summarize them" (triggers many Read calls)
- "Check 5 system statuses and report" (triggers many Bash calls)

The principle: **a long session = a character moving for longer**, because every `pre_tool_call`/`post_tool_call` is one movement. If you send a short task that answers in one word, the character moves a little then goes back to idle.

> 💡 Try sending a long task and leave the office open — you will see the character "really working" clearly.

---

## 🆘 Still stuck?

If you tried everything and are still stuck:

1. Check `sudo systemctl status pixel-office` — is the service running?
2. Check `~/.hermes/config.yaml` — are the hooks all present, is `hooks_auto_accept: true` set?
3. Check `~/.pixel-agents/server.json` — does it exist, and does its port match what the service runs?
4. Trace both logs (item 6)
5. Open an issue on GitHub and attach the logs from `~/.hermes/pixel_agents_bridge.log` and `~/pixel-office.log`

---

Back to [README](../README.md) · [Architecture](ARCHITECTURE.md) · [Plugin vs Bridge](PLUGIN-VS-BRIDGE.md)
