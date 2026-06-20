# 🏢 Hermes Pixel Agents — Watch your Hermes AI agents work live as pixel-art characters

> **Like The Sims, but real.** This project lets you see your Hermes AI agent "walk, sit, type, and talk" inside a pixel-art office right in your browser, while it is actually working (running tools, replying on Telegram, running cron jobs).

Every time a Hermes agent starts a session, calls a tool, or finishes a task, the pixel character that represents it **moves and acts in the office in real time**. You just open your browser and watch — like watching employees work through a window.

This project **connects two worlds**:

- [**NousResearch/hermes-agent**](https://github.com/NousResearch/hermes-agent) (MIT) — the real Hermes agent runtime that runs commands, chats over Telegram/gateway/CLI/cron
- [**pixel-agents-hq/pixel-agents**](https://github.com/pixel-agents-hq/pixel-agents) — the pixel-art office engine
- **The bridge:** we add `HermesBridge` into a fork of pixel-agents, plus a small `pixel_agents_bridge.py` script that translates Hermes shell-hook events into animations

The result: **every Hermes session — CLI, Telegram bot, gateway, cron — shows up as a character that moves in the same office.**

> 📝 **Language note:** This is the English version. Thai versions are also available: [`README.th.md`](README.th.md) and the files in [`docs/th/`](docs/th/).

---

## 📑 Table of Contents

- [How it works](#-how-it-works)
- [Prerequisites](#-prerequisites)
- [Install in 6 steps](#-install-in-6-steps)
- [Open the office](#-open-the-office)
- [Manage the service](#-manage-the-service)
- [Repo file table](#-repo-file-table)
- [Security](#-security)
- [More docs](#-more-docs)
- [Credits & License](#-credits--license)

---

## 🧠 How it works

The big picture — everything travels over **localhost** because the office and Hermes run on the **same machine** (usually a single VPS). No tunnel needed, no API key needed:

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                        ONE VPS (single machine)                  │
 │                                                                  │
 │   ┌───────────────┐    shell hooks        ┌──────────────────┐  │
 │   │  Hermes agent │ ───────────────────▶  │ pixel_agents_     │  │
 │   │  (CLI / TG /  │   (config.yaml)       │ bridge.py         │  │
 │   │   gateway /   │                       │  → HTTP POST      │  │
 │   │   cron)       │                       └────────┬──────────┘  │
 │   └───────────────┘                                │             │
 │                                                    ▼             │
 │                       POST http://127.0.0.1:3100/api/hooks/hermes
 │                                                    │             │
 │                                                    ▼             │
 │                                          ┌──────────────────┐    │
 │                                          │  pixel-office     │    │
 │                                          │  (systemd)        │    │
 │                                          │  ├─ HermesBridge   │    │
 │                                          │  └─ WebSocket /ws  │    │
 │                                          └────────┬──────────┘    │
 └────────────────────────────────────────────────────┼──────────────┘
                                                     │ WebSocket
                                                     ▼
                                          ┌──────────────────────┐
                                          │  Your browser         │
                                          │  pixel office page    │
                                          │  http://<VPS>:3100     │
                                          └──────────────────────┘
```

**The simple step-by-step:**

1. A Hermes agent starts working (someone sends a Telegram message / runs the CLI / a cron time hits)
2. Hermes fires a **shell hook** that we registered in `~/.hermes/config.yaml` (e.g. `on_session_start`, `pre_tool_call`)
3. The shell hook runs `python3 ~/.hermes/pixel_agents_bridge.py` and passes the event payload via stdin/argv
4. The bridge reads `~/.pixel-agents/server.json` to find the office's port+token, converts the event into a standard payload, and **POSTs** to `http://127.0.0.1:3100/api/hooks/hermes`
5. On the office side, `HermesBridge` receives the payload and pushes it over a **WebSocket** to every open browser
6. The character in the office **moves / changes emote / changes spot** based on the event type

> 💡 **The key point:** all of this runs on `127.0.0.1` (localhost) of a single machine — **no SSH tunnel, no API key, nothing to plug in**, because the office and Hermes already live together. Hermes is already authenticated on the VPS, so the office never needs its own credentials.

---

## ✅ Prerequisites

Before you start, check you have all of these:

| Requirement | Note |
|---|---|
| 🖥️ **A machine already running Hermes** | A **VPS** (Ubuntu/Debian) is recommended, because the office needs to run 24/7 alongside Hermes |
| 🔑 **Root or sudo access** | Needed to install the systemd service |
| 🟢 **Node.js 20+** | Check with `node --version`; install it if missing (e.g. via nvm or NodeSource) |
| 📦 **git** | `git --version` |
| 🤖 **Hermes installed and working** | You need `~/.hermes/config.yaml` present and a provider/model set (try running `hermes` and getting a reply) |
| 🌐 **Port 3100** (or your chosen port) open on the VPS | If you want to view from a browser outside the machine — otherwise use an SSH forward (see Security) |

> ⚠️ If Hermes is not installed yet, install Hermes first following the docs for [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent). This project assumes Hermes already runs.

---

## 🚀 Install in 6 steps

There are two paths: **(A) one-command script** for speed, or **(B) manual step-by-step** for understanding.

### Path A — One-command install (recommended)

```bash
git clone https://github.com/aiunlocked1412/hermes-agent-pixel.git
cd hermes-agent-pixel
bash scripts/install.sh
```

This script does everything automatically:

1. **Clones/builds** the pixel-agents fork (the one that includes `HermesBridge`) into the right location
2. **Builds** the office (Next.js/Node) so it is ready to run
3. **Installs the bridge** (`pixel_agents_bridge.py`) to `~/.hermes/pixel_agents_bridge.py`
4. **Installs the shell hooks** into your `~/.hermes/config.yaml` (auto-merged, it never destroys your existing file)
5. **Installs the systemd service** (`pixel-office.service`) so the office runs as a background service and wakes up on every boot
6. **Health check** — fires a GET at the office and reports whether it works

When it finishes, jump straight to [Open the office](#-open-the-office).

---

### Path B — Manual step-by-step (for full control)

If you do not want to trust the script, or your machine has a special setup, follow along:

#### Step 1 — Clone the pixel-agents fork and build

```bash
git clone https://github.com/aiunlocked1412/hermes-agent-pixel.git
cd hermes-agent-pixel
npm install
npm run build
```

#### Step 2 — Install the bridge script

Copy `pixel_agents_bridge.py` into the Hermes home directory:

```bash
cp pixel_agents_bridge.py ~/.hermes/pixel_agents_bridge.py
chmod +x ~/.hermes/pixel_agents_bridge.py
```

This file receives the event from the Hermes shell hook via stdin, reads `~/.pixel-agents/server.json` (written by the office at start) to find the `port` and `token`, then POSTs to `/api/hooks/hermes`.

#### Step 3 — Add shell hooks to `~/.hermes/config.yaml`

The example file is at `config/hermes-hooks.yaml.snippet` in the repo. Open your `~/.hermes/config.yaml` and **merge** the following `hooks:` block into it (if a `hooks:` key already exists, merge into the same key — do not delete the old one):

```yaml
hooks:
  on_session_start:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  on_session_end:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  pre_tool_call:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  post_tool_call:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  subagent_start:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  subagent_stop:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
hooks_auto_accept: true
```

What each hook does:

| Hook | When it fires | Effect in the office |
|---|---|---|
| `on_session_start` | the agent starts a session | the character "wakes up" / enters the office |
| `on_session_end` | the agent ends a session | the character goes to sit / idle |
| `pre_tool_call` | before calling a tool (Bash, Read, …) | the character moves to a machine / desk |
| `post_tool_call` | after a tool returns a result | the character changes pose / emote |
| `subagent_start` | a subagent is spawned | a subagent character appears |
| `subagent_stop` | a subagent finishes | the subagent character leaves |

> 💡 `hooks_auto_accept: true` is very important — without it, Hermes will ask for confirmation before every hook fires, and the office will stutter.

#### Step 4 — Install the systemd service

Copy `pixel-office.service` into systemd:

```bash
sudo cp pixel-office.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now pixel-office
```

This service runs the office (Node) on port 3100 and writes logs to `~/pixel-office.log`.

#### Step 5 — Health check

```bash
curl http://127.0.0.1:3100/api/health
# expect 200 OK
```

#### Step 6 — Restart Hermes once (important)

Sessions that started **before** the hooks were installed **will not send events** (because config is loaded at start). Restart the gateway/profile once so it picks up the new hooks:

```bash
sudo systemctl restart hermes-gateway-watchdog   # or restart the profile your way
```

After this, every new session will send events.

---

## 👀 Open the office

Open your browser at:

```
http://<YOUR_VPS_IP>:3100
```

Then use Hermes normally — send a Telegram message, run the CLI, or wait for cron. The characters will start to move.

### Character rules (important — read before you panic)

- 🧍 **One character per profile** — each Hermes profile (`trader`, `telegram`, `default`, …) is a **single persistent character** for its whole lifetime, not a new one for every message
- 🪑 **Characters "sit there" first** — they only move when there is activity (calling a tool / replying); while idle they just sit, they do not vanish
- 🔄 **Persistent** — the character's ID is computed from the profile name (uuid5), so it is always the same character even after restarting the office
- 🎭 **Subagents** have their own characters, appearing/leaving on `subagent_start`/`subagent_stop`

> If you open the page and "nothing moves" — do not panic. The character is probably idle. Try sending a task that makes Hermes call several tools, then look again.

---

## 🛠️ Manage the service

The office runs as a systemd service named `pixel-office`:

```bash
# view status
sudo systemctl status pixel-office

# restart
sudo systemctl restart pixel-office

# stop
sudo systemctl stop pixel-office

# live logs
tail -f ~/pixel-office.log
```

### Change port / host

Edit `/etc/systemd/system/pixel-office.service` — find the line that sets `PORT` or `HOST` (in the `Environment=` section), e.g.:

```ini
Environment=PORT=3200
Environment=HOST=127.0.0.1
```

Then reload:

```bash
sudo systemctl daemon-reload
sudo systemctl restart pixel-office
```

> If you change the port, remember to also update the URL in `~/.hermes/pixel_agents_bridge.py` (or set `PIXEL_OFFICE_PORT`) and the URL in your browser — but normally the bridge reads the port from `server.json` automatically.

---

## 📁 Repo file table

| File / Folder | Purpose |
|---|---|
| `scripts/install.sh` | The automatic installer (Path A) |
| `pixel_agents_bridge.py` | **The heart** — receives Hermes shell-hook events and POSTs them to the office |
| `config/hermes-hooks.yaml.snippet` | The `hooks:` block to merge into `~/.hermes/config.yaml` |
| `pixel-office.service` | systemd unit file to run the office as a service |
| `server/` | Server-side source of pixel-agents (includes `HermesBridge`) |
| `server/src/hermesBridge.ts` | The code that receives `/api/hooks/hermes` and broadcasts over WebSocket |
| `src/` | Browser-side source (the pixel-art office page) |
| `docs/ARCHITECTURE.md` | Deep architecture explanation |
| `docs/PLUGIN-VS-BRIDGE.md` | Why we use the bridge instead of the plugin |
| `docs/TROUBLESHOOTING.md` | Common problems + fixes |
| `fixes/` | Platform-specific fix notes (e.g. Windows build) |

---

## 🔒 Security

### Default: `0.0.0.0` = anyone can view the office

By default the office binds to `0.0.0.0`, meaning **anyone who knows your VPS IP can open and view the office page**. But:

- ✅ They only see **activity** (characters moving) — they cannot see conversation content or secrets
- 🔐 **Injecting events** (POSTing to `/api/hooks/hermes`) requires the **token** stored in `server.json` — outsiders who do not know the token cannot push fake events in

### If you want to restrict it: bind `127.0.0.1` + an SSH forward tunnel

Edit `pixel-office.service`:

```ini
Environment=HOST=127.0.0.1
```

Then restart. Now the office is only reachable from inside the VPS itself. When you want to view it from your laptop, use an **SSH forward tunnel** (the only safe way):

```bash
ssh -L 3100:127.0.0.1:3100 <user>@<YOUR_VPS_IP>
```

Then open `http://127.0.0.1:3100` on your laptop — the traffic flows encrypted through SSH.

> ⚠️ **Never use a reverse tunnel (`ssh -R`)** with this project — it usually dies with "remote port forwarding failed" and is not needed at all, because the office already runs on the same machine as Hermes. See `docs/TROUBLESHOOTING.md` for details.

### Files you must never commit

Make sure your `.gitignore` covers:

- `~/.pixel-agents/server.json` — contains the office token
- `.env` — contains Hermes secrets
- `id_rsa` / SSH key material

Do not accidentally push these to GitHub.

---

## 📚 More docs

- 📐 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — deep architecture (HermesBridge, uuid5, discovery, why not the Claude path)
- 🧩 [`docs/PLUGIN-VS-BRIDGE.md`](docs/PLUGIN-VS-BRIDGE.md) — why we use the bridge, not the `pixel_observer` plugin
- 🩹 [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) — real problems + fixes

---

## 🙏 Credits & License

This project stands on the shoulders of two community projects:

- **[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)** (MIT) — the real Hermes agent runtime
- **[pixel-agents-hq/pixel-agents](https://github.com/pixel-agents-hq/pixel-agents)** — the pixel-art office engine
- **[aiunlocked1412/hermes-agent-pixel](https://github.com/aiunlocked1412/hermes-agent-pixel)** — the fork that adds `HermesBridge` and the bridge script to connect the two

**License:** MIT — use, modify, and distribute freely.

---

> Made with care 💚 so everyone can see their own agent "come alive". If you hit a problem, read [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) first, then open an issue.
