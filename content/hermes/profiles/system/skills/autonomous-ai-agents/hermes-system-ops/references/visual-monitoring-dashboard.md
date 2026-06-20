# Visual Monitoring Dashboard Pattern

Combining a Python HTTP status API with a p5.js (or plain HTML/Canvas) visual
frontend to create a real-time monitoring dashboard for a Hermes deployment.

## Architecture

```
Browser (visual frontend)
  ↓ fetch /api/status every 4s
Python HTTP Server (port 9120)
  ├── Serves static HTML/JS from /root/pixel-agent-office/
  └── /api/status → JSON with live backend health
       ├── agents[]     — pgrep + ps for each gateway profile
       ├── docker        — docker ps --filter
       ├── dashboard     — curl localhost:9119
       ├── watchdog      — pgrep -f gateway-watchdog.sh
       └── system        — /proc/loadavg, free, df, /proc/uptime
```

## When to Use

- User wants a **visual** representation of agent status (pixel art, retro UI, etc.)
- User says "อยากเห็น" (want to see) the backend — they want a dashboard, not just text
- Pairing with `delegate_task` to a Coder subagent for the visual frontend while
  the System Agent builds the backend status API

## Workflow

1. **Build the status API server** — Use `templates/status-api-server.py` as a
   starting point. It serves both static files and the `/api/status` JSON endpoint.
   Key checks: `pgrep` for gateways, `docker ps` for containers, `curl` for
   dashboard, `/proc/` for system stats.

2. **Delegate the visual frontend** to a Coder subagent with:
   - Clear spec of what to draw (agents, desks, screens, animations)
   - The JSON API shape so the frontend knows what to fetch
   - `toolsets: ["terminal", "file"]` so it can write the HTML file
   - Instructions to use p5.js from CDN (single self-contained HTML)

3. **Start the server** — `python3 server.py --port 9120 --directory /path/to/html`
   Use `terminal(background=true)` since it's a long-running server.

4. **Verify** — `curl http://localhost:9120/api/status | python3 -m json.tool`
   to confirm JSON, then `browser_navigate` to confirm the page renders.

5. **Share with user** — Give the public URL: `http://<server-ip>:9120/index.html`

## JSON Status Shape

```json
{
  "agents": [
    {"name": "trader", "status": "online", "pid": 12345, "cpu": 2.1, "mem": "321MB"}
  ],
  "docker": {"name": "claw-trade-mt5", "status": "running", "uptime": "3h"},
  "dashboard": {"status": "online", "port": 9119, "http_code": 302},
  "watchdog": {"status": "running", "pid": 2737401},
  "system": {"cpu_load": "0.57", "mem_used": "2.2G/11G", "disk_used": "36G/193G", "uptime": "5d 20h"},
  "timestamp": "2026-06-19 17:27:02"
}
```

## Delegate Task Pattern for Visual Frontend

When delegating pixel-art / p5.js dashboard work to a subagent:

```
goal: "Create a self-contained HTML file for [description]..."
context: "API at http://localhost:PORT/api/status, JSON shape: {...}"
toolsets: ["terminal", "file"]
```

Key instructions to include:
- Single self-contained HTML (p5.js from CDN, no external images)
- Canvas size (1280x720 works well)
- Fetch API status every 4s with graceful fallback (all-green default)
- Pixel-art drawn programmatically with `rect()` calls
- Status indicators that change color based on API data
- Mobile responsive (canvas scales to viewport)
- Retro pixel font from Google Fonts ("Press Start 2P", "VT323")

## Pitfalls

### Terminal background command for server startup

Hermes terminal blocks `&` backgrounding in foreground mode. Use
`terminal(background=true, notify_on_complete=true)` to start the HTTP server,
then verify with a separate `curl` call.

### Dashboard build delay (already in SKILL.md)

`hermes dashboard` takes 20-30s to build web UI before port 9119 listens.
Don't kill and restart during build — wait for `HERMES_DASHBOARD_READY` in log.

### p5.js canvas screenshot limitation

Browser tools cannot directly screenshot a p5.js canvas as an image file.
Use `browser_snapshot` (accessibility tree) and `browser_console` to verify
content is rendering. Check `canvas.toDataURL()` length > 1000 to confirm
the canvas has content.

### CORS for status API

The Python HTTP server must set `Access-Control-Allow-Origin: *` on the
`/api/status` endpoint, or the browser will block the fetch if the HTML
is served from a different origin.