# Trading Bot Retro Dashboard — Full Working Example

This is a concrete example from a session where the user asked for a "pixel art game-style dashboard" for their XAU/USD gold trading bot. The dashboard shows real trading data from a SQLite database, agent statuses, and monthly P&L charts.

## Source Files

The full implementation lives at `/root/Claw_Trade/dashboard_server.py` and was deployed on AWS EC2 at port 8080.

## Architecture

```
dashboard_server.py (single file — Python backend + embedded HTML)
│
├── DashboardAPI class ─── reads from:
│   ├── trade_memory.db (SQLite - trade history)
│   ├── learned_config.json (parameter overrides)
│   └── ps aux / docker ps (bot + container status)
│
├── DashboardHandler ─── HTTP routes:
│   ├── GET /          → retro HTML page (DASHBOARD_HTML)
│   ├── GET /api/overview → stats + trades + breakdowns
│   ├── GET /api/agents   → agent statuses + thresholds
│   └── GET /api/chart    → equity curve data
│
├── DASHBOARD_HTML ─── single-page SPA (CSS + JS embedded):
│   ├── Header: bot status dot, balance, live indicator
│   ├── Stats Grid: 4 cards (total trades, wins, losses, P&L)
│   ├── Performance: win rate circle, avg R:R, total trades
│   ├── Monthly P&L: retro bar chart (green/red bars)
│   ├── AI Agents: 6 character cards with HP bars + levels
│   ├── By Regime: per-market-regime win rate table
│   ├── By Session: per-trading-session breakdown
│   └── Recent Trades: full table with signal badges
└── Starts on port 8080
```

## API Endpoint Examples

### GET /api/overview

```json
{
  "status": "ok",
  "total_trades": 2,
  "wins": 1,
  "losses": 1,
  "winrate": 50.0,
  "total_pnl": 265.0,
  "avg_r": 0.71,
  "consecutive_losses": 0,
  "monthly": [
    {"month": "2026-06", "trades": 2, "wins": 1, "losses": 1, "pnl": 265.0}
  ],
  "by_regime": [
    {"regime": "RANGING", "total": 1, "wins": 0, "pnl": -80.0, "winrate": 0.0},
    {"regime": "TRENDING", "total": 1, "wins": 1, "pnl": 345.0, "winrate": 100.0}
  ],
  "by_session": [...],
  "by_confidence": [...],
  "recent_trades": [...]
}
```

### GET /api/agents

```json
{
  "agents": [
    {
      "name": "🧙 Quant Analyst",
      "title": "เทคนิคัล วิซาร์ด",
      "status": "active",
      "emoji": "🔮",
      "color": "#00ff88",
      "duty": "วิเคราะห์ RSI, MACD, EMA, Fibo",
      "level": 8,
      "hp": 92
    }
  ],
  "bot_running": true,
  "container_status": "Up 5 hours"
}
```

## Color Palette

| Purpose | Hex | Usage |
|---------|-----|-------|
| Background dark | `#0a0a1a` | Page background |
| Panel dark | `#111128` | Card backgrounds |
| Border | `#2a2a5a` | Panel borders |
| Text | `#c8d6e5` | Primary text |
| Dim text | `#576574` | Secondary text |
| Green (win) | `#00ff88` | Positive P&L, win stats |
| Red (loss) | `#ff4757` | Negative P&L, loss stats |
| Gold | `#ffd700` | Titles, accents, sparks |
| Blue | `#4488ff` | Secondary accent |

## Deployment Commands

```bash
# Start server in background
python3 dashboard_server.py &

# Or with background process tracking
terminal(command="cd /path/to/project && python3 dashboard_server.py", background=true)

# Open port in firewall
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
iptables -I FORWARD -p tcp --dport 8080 -j ACCEPT

# Block a port
iptables -A INPUT -p tcp --dport 3000 -j DROP
iptables -A FORWARD -p tcp --dport 3000 -j DROP
```

## Security Notes

- The Python `http.server` module has no built-in auth. For internal/team dashboards, add a simple token check in `do_GET`, or wrap with nginx basic auth.
- The `Access-Control-Allow-Origin: *` header exposes the API to any website. Remove or restrict it for production.
- iptables rules are ephemeral (reset on reboot). Persist them with `iptables-save > /etc/iptables/rules.v4` or use `ufw`.

## Key CSS Selectors for Customization

- `.header-title` — brand text in Press Start 2P
- `.stat-card .number` — large stat numbers (gold/green/red classes)
- `.bar.green` / `.bar.red` — monthly P&L bars
- `.agent-card:hover` — glow effect on hover
- `.wr-circle` — win rate circular display
- `.loss-alert` — pulsing red alert for consecutive losses
- `.live-dot.on` — green pulsing dot for bot status