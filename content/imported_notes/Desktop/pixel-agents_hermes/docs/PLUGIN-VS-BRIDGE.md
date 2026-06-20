# 🧩 Plugin vs Bridge — the most "painful" lesson

> This document is a summary of the problem that ate the most time on this project. Read it to the end before choosing an approach.

---

## TL;DR (one line)

> **Use the bridge (shell hooks). Do not insist on the `pixel_observer` plugin.** The plugin works for the CLI, but **Hermes Telegram/gateway does not fire plugin hooks**, so if you use the plugin you will usually not see characters on Telegram.

---

## 📜 Intro: pixel-agents has two entry points

pixel-agents (upstream) was originally designed for Claude Code using the Python runtime's plugin (e.g. `pixel_observer`). When moving to Hermes, we had two choices:

| Option | Mechanism | Works for CLI | Works for Telegram/gateway |
|---|---|---|---|
| **A. `pixel_observer` plugin** (Python) | registers as a Hermes Python runtime plugin, receives `pre_tool_call`/`post_tool_call` plugin hooks | ✅ yes | ❌ **no** |
| **B. Bridge script** (shell hooks) | registers shell hooks in `~/.hermes/config.yaml` → Hermes spawns `pixel_agents_bridge.py` | ✅ yes | ✅ **yes** |

---

## 🤒 The symptom (why we chose the bridge)

At first we tried option A (the plugin) because it looked "cleaner" — no need to touch config.yaml, no process to spawn. But then we hit this:

### Symptom

- ✅ When running **`hermes` in the CLI** → the character appears in the office and moves normally
- ❌ When using the **Telegram bot** or **gateway** → the character **does not move at all**, even though Hermes replies to messages fine

In short: **CLI shows up, Telegram does not.**

---

## 🔬 The root cause (after digging)

After tracing the code and logs, we found that Hermes has **"two separate hook sets"**:

### 1) Plugin hooks (for the Python runtime)

The `pixel_observer` plugin registers with the **Hermes Python runtime**, only when a CLI session (interactive) is spawned. So:

- ✅ CLI session → runtime starts → plugin loads → plugin hooks fire → character moves
- ❌ Telegram/gateway → **does not go through the same Python runtime path** → plugin hooks are never fired

### 2) Shell hooks (from `config.yaml`)

Shell hooks set in `~/.hermes/config.yaml` are a **core-level mechanism** that Hermes fires from every source — whether it comes from CLI, Telegram, gateway, or cron — because they are part of the session's main lifecycle.

> 💡 Think of it simply: **a plugin is an add-on that some runtimes load; a shell hook is a bell Hermes rings every time.**

### Extra reasons the plugin fails for Telegram

1. **The gateway starts before the plugin is enabled** — often the gateway/watchdog starts at boot, before the plugin is enabled/detected, so the plugin is not in the gateway's mind from the start
2. **The gateway does not load plugin hooks** — even if you enable the plugin later, the gateway usually does not re-scan and does not load plugin hooks into its pipeline, so Telegram events never pass through the plugin

Net result: the plugin **only works for the CLI** in our setup.

---

## ✅ The fix we actually use: the bridge (shell hooks)

Instead of fighting the plugin lifecycle, we picked the path Hermes guarantees to fire everywhere:

1. Install the shell hooks in `~/.hermes/config.yaml` (see `config/hermes-hooks.yaml.snippet`)
2. Every hook calls `python3 ~/.hermes/pixel_agents_bridge.py`
3. The bridge converts and POSTs to the office (see `ARCHITECTURE.md` for detail)

Result:

- ✅ CLI → moves
- ✅ Telegram → moves
- ✅ gateway → moves
- ✅ cron → moves

All of this because the shell hook is the one mechanism Hermes fires from **every source**.

---

## 🧪 If you still want to try the plugin (you can, but be careful)

If anyone wants to experiment with the plugin path, you can — the file lives at `hermes-plugin/pixel_observer` in the upstream fork. Rough steps:

```bash
# (example — adjust to your upstream)
cd hermes-plugin/pixel_observer
# install it the way Hermes plugins are installed
hermes plugins install ./pixel_observer
# or follow the plugin-enable mechanism of your Hermes version
```

**But remember: it will not work for Telegram/gateway** (for the reasons above). It only works when running `hermes` in the CLI.

### When is the plugin worth considering?

- 🧪 Experimenting/studying on your own machine, not caring about Telegram
- 🎯 You want a specific field that only exists in the Python runtime (e.g. internal state)

If your goal is "watch the agent work from every channel," **the bridge is the only answer.**

---

## 📊 Summary comparison table

| Dimension | `pixel_observer` plugin | Bridge (shell hooks) |
|---|---|---|
| CLI | ✅ | ✅ |
| Telegram | ❌ | ✅ |
| Gateway | ❌ | ✅ |
| Cron | ❌ (runtime dependent) | ✅ |
| Setup | enable plugin | merge hooks in `config.yaml` |
| Process overhead | in-process with the runtime | spawns python per event (small) |
| Cross-source stability | low | **high** |
| Recommendation | CLI experiments only | ✅ **for real use** |

---

## 🎯 Conclusion

- **The problem:** the `pixel_observer` plugin works for the CLI but does not move on Telegram
- **The cause:** Hermes gateway/Telegram fires only shell hooks, not plugin hooks; the gateway usually starts before the plugin is enabled/loaded
- **The fix:** use the **bridge script** registered as **shell hooks** instead — guaranteed by Hermes to fire from every source

> Do not waste time fighting the plugin if your goal is to watch Telegram — use the bridge from the start and save yourself hours.

---

Back to [README](../README.md) · [Architecture](ARCHITECTURE.md) · [Troubleshooting](TROUBLESHOOTING.md)
