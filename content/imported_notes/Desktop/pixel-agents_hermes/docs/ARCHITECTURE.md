# 📐 Deep Architecture — Hermes Pixel Agents

This document explains how every piece connects. It is for people who want to modify, extend, or understand what is under the hood.

---

## 🧩 The main components (4 pieces)

```
┌──────────────┐   ┌────────────────┐   ┌────────────────────┐   ┌──────────────┐
│   Hermes     │──▶│  Bridge script │──▶│   pixel-office     │──▶│   Browser    │
│  (runtime)   │   │  (Python)      │   │  (Node + systemd)  │   │  (pixel UI)  │
│              │   │                │   │  ├─ HermesBridge   │   │              │
│  shell hooks │   │  reads         │   │  └─ WebSocket /ws  │   │              │
│  in config   │   │  server.json   │   │                    │   │              │
└──────────────┘   └────────────────┘   └────────────────────┘   └──────────────┘
  event sender      converter+poster      receiver+broadcaster   renderer
```

### 1) Hermes runtime (sender side)

The Hermes agent is already running on the machine (VPS). We add **shell hooks** into `~/.hermes/config.yaml`. Every time an event happens (`on_session_start`, `pre_tool_call`, …), Hermes spawns the command we specify and sends the payload (JSON) via stdin.

> Key fact: the Hermes **gateway/Telegram fires only shell hooks** — it does not fire the Python runtime's plugin hooks. This is why we need the bridge (see [`PLUGIN-VS-BRIDGE.md`](PLUGIN-VS-BRIDGE.md)).

### 2) The bridge script (`pixel_agents_bridge.py`)

A small Python script that does three things:

1. **Reads the event payload** from the stdin that Hermes sends
2. **Finds the office** by reading `~/.pixel-agents/server.json` — this file stores the office's `port` and `token` (written by the office at start)
3. **POSTs** to `http://127.0.0.1:<port>/api/hooks/hermes` with an `Authorization` header carrying the token

The bridge **re-reads `server.json` every time** it is called, so even if the office restarts and the token changes, nothing needs reconfiguring.

### 3) pixel-office (receiver + broadcaster)

The pixel-agents fork where we added `HermesBridge`. It runs as a **systemd service** named `pixel-office`.

- **Listens for HTTP** at `POST /api/hooks/hermes` — the entry point for events from the bridge (requires the correct token)
- **Listens for WebSocket** at `/ws` — the channel that pushes data to every browser that has the office open
- **Serves the web page** (pixel-art) at `/`

When `HermesBridge` receives an event, it converts it into a message for the UI and broadcasts it over `/ws` to every client.

### 4) Browser (renderer)

The browser opens `http://<VPS>:3100` and connects a WebSocket to `/ws`. Every message that arrives drives a pixel character (changing pose, moving it, or changing its emote).

---

## 🆔 Why "one character per profile" — the uuid5 story

Each character needs a **stable ID for its whole lifetime** — not a new one for every message, or the office would fill up with duplicates.

We use **uuid5** (a UUID derived deterministically from a name + namespace hash):

```python
character_id = uuid5(NAMESPACE, profile_name)
# e.g. the "telegram" profile → always the same ID, no matter how many messages
```

Results:

- ✅ **Persistent** — the `trader` profile is always the same character, even after restarting the office or VPS
- ✅ **Stable** — the character "sits and waits" first, and only moves on activity; it does not spawn/die on every event
- ✅ **Multi-profile** — `telegram`, `trader`, `default`, … each gets its own character, all in the same office at the same time
- ✅ **Subagents** follow the same rule — a subagent of a given profile has its own ID and appears/leaves on `subagent_start`/`subagent_stop`

> If you rename a profile, its ID changes, so it counts as a new character (the old one keeps "sitting" until it hits the idle timeout).

---

## 🔍 Discovery — how the bridge finds the office

The office does not use a fixed port, because it might clash with another service. Instead, at start it writes a "name card" into a file:

**`~/.pixel-agents/server.json`**

```json
{
  "port": 3100,
  "token": "<random-token-written-at-startup>",
  "host": "127.0.0.1"
}
```

The bridge reads this file every time, so it always knows both the port and the token, and POSTs to the right place.

> 💡 Bonus: if you change the port in the service, or the office restarts and hands out a new random token, **the bridge follows along automatically**. No config edit, no Hermes restart.

---

## 🤔 Why `/api/hooks/hermes` and not `/api/hooks/claude`

The upstream pixel-agents was originally designed for Claude Code — it has an `/api/hooks/claude` endpoint mapped to a specific Claude runtime path (e.g. `SessionStart`, `PreToolUse`, with Claude-specific field structures).

We tried the Claude path with Hermes and it **rendered poorly**, because:

| Problem | Detail |
|---|---|
| 🧩 Schema mismatch | Hermes event fields differ from Claude (tool names, session structure) → characters glitch or move out of rhythm |
| 🎭 No Telegram/gateway view | The Claude path was designed for the Claude CLI only, with no concept of multiple sources (Telegram, cron) |
| 🔁 Duplicate characters | The Claude path uses ID logic that does not match the "profile" concept in Hermes → the same character might be created multiple times |

So we decided to **add a new endpoint `/api/hooks/hermes`** with a `HermesBridge` designed specifically for Hermes's event schema:

- ✅ Maps Hermes fields → office actions correctly
- ✅ Uses uuid5 per-profile → stable characters
- ✅ Supports multiple sources (CLI/Telegram/gateway/cron) because they all fire the same shell hook

> The upstream Claude path is still there (untouched) — anyone using Claude Code can still use pixel-agents as normal. We just added a Hermes option.

---

## 🔄 End-to-end event lifecycle

Follow a `pre_tool_call` event from origin to screen:

```
1. Hermes is about to call a tool (e.g. Bash)
        │
        ▼
2. Hermes fires the "pre_tool_call" shell hook
   → spawns: python3 ~/.hermes/pixel_agents_bridge.py
   → sends a JSON payload via stdin
   { "hook": "pre_tool_call", "profile": "telegram",
     "tool": "Bash", "session_id": "abc123", ... }
        │
        ▼
3. The bridge reads stdin
   → reads ~/.pixel-agents/server.json  (gets port=3100, token=<...>)
   → converts the payload to the HermesBridge format
   → POSTs http://127.0.0.1:3100/api/hooks/hermes
      Authorization: Bearer <token>
        │
        ▼
4. pixel-office receives it at HermesBridge
   → validates the token ✓
   → computes character_id = uuid5(NAMESPACE, "telegram")
   → builds a message: { character_id, action: "use_tool", tool: "Bash", ... }
   → broadcasts over WebSocket /ws
        │
        ▼
5. Every open browser receives the message
   → the pixel engine drives the "telegram" character to the Bash machine
   → changes the animation to "typing / working"
        │
        ▼
6. (later) post_tool_call arrives → the character changes pose to "done"
```

All of this takes < 100ms because it travels over localhost.

---

## 📊 Endpoint summary

| Endpoint | Method | Who calls it | What it does |
|---|---|---|---|
| `/` | GET | browser | serves the pixel-art office page |
| `/ws` | WebSocket | browser | receives real-time pushed events |
| `/api/hooks/hermes` | POST (token required) | the bridge script | receives a Hermes event → broadcasts |
| `/api/hooks/claude` | POST | Claude Code (upstream) | receives a Claude event (exists, unused for Hermes) |
| `/api/health` | GET | anyone | health check (200 OK) |

---

## 🔧 Extension points

- **Add a new event type:** edit the bridge to send an extra field, then edit `server/src/hermesBridge.ts` to map it to a new action
- **Change the uuid namespace:** edit the constant in both the bridge and `hermesBridge.ts` (do them together, or the IDs will not match)
- **Support multiple machines:** currently designed for a single machine. If you want to split Hermes and the office onto different machines, change the bridge to POST across the network (open the port + handle the token/HTTPS yourself)

---

Back to [README](../README.md) · [Plugin vs Bridge](PLUGIN-VS-BRIDGE.md) · [Troubleshooting](TROUBLESHOOTING.md)
