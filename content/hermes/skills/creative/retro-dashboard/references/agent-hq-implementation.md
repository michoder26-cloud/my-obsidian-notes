# Agent HQ — Canvas Pixel Agent Dashboard (June 2026)

## Context

Built for a Thai-speaking user (TK TF) running a Claw_Trade XAU/USD trading bot on AWS EC2. The user wanted:

- **Multiple pages** (not single-page): Dashboard, Agent HQ, Trades, Settings
- **Agent HQ page**: pixel-art characters walking around an office, real-time activity visualization — inspired by a YouTube video about "21 AI agents running a coffee shop" (NokKung channel) and the pixel-agents VS Code extension (github.com/pixel-agents-hq/pixel-agents)
- **Dashboard page**: modern/clean UI (not pixel), stats + charts
- **Real data** from live SQLite trade database

## Design Variants

### Variant 1: Dark Cyber Palette (Initial)

See main SKILL.md "Dark Cyber Palette" for colors. Used for the first version — dark backgrounds, cyan/green accents, CRT aesthetic.

### Variant 2: BBR Warm Palette (Final — user preferred)

The user provided reference images of the "BrewBerich Co., Ltd." (BBR) pixel agent dashboard and wanted it matched exactly. Key differences:

**Color scheme**: warm browns, golds, wood tones instead of dark blue/cyan:
- Floor: `#8a7a4a` / `#9a8a5a` checkerboard (was `#1a1a3e`)
- Desks: `#958a6a` wood grain (was `#3a3a6a`)
- Monitors: `#38ba72` CRT green (was `#111` with activity glow)
- Characters: `#e8d08a` golden bodies (was colored by agent type)
- Terminal BG: `#c8b888` beige (was `#0a0a14` dark)

**Layout**: split-view with 62% pixel office / 38% status panel (was 65/35).

**Status panel**: replaced the retro-green-terminal look with a warm beige panel with #46352d header, agent list split by Working/Idle groups.

**Dashboard**: replaced dark metric cards with warm white `#fff8ee` cards on `#f5f0e0` page background.

**Furniture**: added detailed CRT monitors with green phosphor, wood-grain desk textures, chairs with backs, and potted plants in corners — all absent from the dark cyber version.

**File**: `/root/Claw_Trade/agent_hq_server.py` was rewritten to use the BBR palette. See `references/bbr-palette.md` for exact color measurements.

## Source File

`/root/Claw_Trade/agent_hq_server.py` — single-file Python server (~33KB, ~700 lines)

## Pages

| URL | Purpose | Style |
|-----|---------|-------|
| `/` | Trading overview — 4 stat cards, monthly P&L bar chart, regime breakdown, recent trades table | Modern (no pixel) |
| `/agents` | Pixel art office with 6 animated AI agents walking around | Canvas pixel art |
| `/trades` | Full trade history table | Clean data table |
| `/settings` | System status, learned parameters, portfolio info | Config panel |

## API Endpoints

| Endpoint | Response |
|----------|----------|
| `GET /api/overview` | Total trades, winrate, P&L, monthly data, regime breakdown, recent trades |
| `GET /api/agents` | 6 agent objects (name, emoji, color, level, hp, duty, activity) + bot status + container status + learned config thresholds |
| `GET /api/chart` | Equity curve array + current balance + growth % |

## Canvas Characters

Six agents rendered as pixel-art characters (16x24 pixel sprites drawn via canvas primitives):

| Agent | Emoji | Color | Level | HP |
|-------|-------|-------|-------|----|
| Quant Analyst | 🔮 | `#00ff88` | 8 | 92 |
| News Analyst | 📡 | `#ffaa00` | 5 | 78 |
| Bull Agent | ⚔️ | `#ff4757` | 7 | 85 |
| Bear Agent | 🛡️ | `#4488ff` | 7 | 88 |
| CEO Agent | 👑 | `#ffd700` | 10 | 99 |
| Learning Engine | 🧬 | `#ff66ff` | 6 | 95 |

Each character has:
- Walking animation (4-frame leg cycle)
- Directional movement toward desk targets
- Activity states: analyzing, reading, typing, deciding, learning, walking, idle
- Activity timer (random switching every 100-300 frames)
- Speech bubble with activity icon above head
- HP bar and level badge

## Activity Log

A real-time log panel on the right shows agent actions with timestamps:

```
[14:32:15] 🔮 Quant Analyst: analyzing market data...
[14:32:17] 👑 CEO Agent: evaluating trade setup...
[14:32:20] 🧬 Learning Engine: optimizing parameters...
```

## Office Environment

Canvas-rendered office with:
- Dark grid floor (`#1a1a3e` with `#2a2a5a` grid lines)
- Wall borders (8px thick)
- Room dividers (2 dividers splitting 3 columns)
- 6 desks with monitors showing activity glow
- Name labels above each desk

## Nav Bar

Consistent across all pages:
```
GOLD SNIPER AI | Dashboard | Agent HQ | Trades | Settings | [● live dot]
```

The live dot (green pulsing/blinking) indicates whether the bot process is running, checked via `/api/agents` endpoint every 10 seconds.

## Deployment

```bash
# Start the server
cd /root/Claw_Trade
python3 agent_hq_server.py &  # Runs on port 8080

# Firewall
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT

# Access
http://<server-ip>:8080/
```

## Auto-Refresh

- Dashboard: 15 seconds
- Agent HQ status/log: 30 seconds
- Trades: 30 seconds
- Settings: 30 seconds
- Bot status dot (nav): 10 seconds

## Pitfalls

- **Single-threaded http.server**: Fine for <5 concurrent users. For more, switch to Flask/gunicorn.
- **Canvas must be visible**: The agent canvas animation pauses if the Agent HQ page isn't open (browser tab throttling). This is expected — it's a dashboard, not a server-side simulation.
- **Font dependency**: Uses system sans-serif for most text. The `Press Start 2P` pixel font is NOT used here (user wanted the dashboard non-pixel; only agents are pixel-art).
- **API data staleness**: Agents randomly switch activities for visual interest, but their activity is NOT tied to real actions from the trading bot. The stats panel on the right refreshes from the real API.