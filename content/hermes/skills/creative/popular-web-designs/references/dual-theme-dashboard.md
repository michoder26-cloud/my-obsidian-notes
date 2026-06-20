# Multi-Theme Dashboard Serving Pattern

When building dashboards in Python HTTP servers that need **multiple themed versions** (e.g. light theme for Dashboard, pixel art for Agent HQ), serve them from the same server using URL/query-parameter routing.

## Why

- Single Python process handles everything (simple deployment, cron-free)
- API endpoints are shared between themes (no duplication)
- User can bookmark or link directly to their preferred theme
- Avoids needing separate servers/ports

## Pattern

```python
class DashboardHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Parse query params BEFORE stripping path
        full_path = self.path
        if "?" in full_path:
            path, query = full_path.split("?", 1)
            theme = dict(q.split("=") for q in query.split("&") if "=" in q).get("theme", "")
        else:
            path = full_path
            theme = ""

        # API routes (shared across themes)
        if path == "/api/overview":
            return self._serve_json(self.api.get_overview())
        # ... more API routes ...

        # Serve HTML based on theme
        if theme == "pixel":
            self._serve_html(DASHBOARD_PIXEL_ART)  # embed as string or load from file
        else:
            self._serve_light_html()  # load from static/index.html or separate constant
```

## URL Routing

| URL | What it shows |
|-----|--------------|
| `/` | Default theme (usually light/clean) |
| `/?theme=pixel` | Pixel art / retro theme |
| `/?theme=dark` | Dark theme variant |

## Embed vs Load from File

| Approach | When |
|----------|------|
| Inline constant `r"""..."""` | Single themed page, < 1000 lines |
| `open("static/index.html")` (file) | Larger pages, needs separate maintenance |
| Both (mixed) | Two distinct themes — embed one, load file for other |

## Pitfalls

- **Parse query BEFORE stripping path**: `self.path` contains `?query`. Split first, extract params, then use path for routing.
- **Static assets**: Use relative paths in HTML so they resolve against same origin regardless of theme URL.
- **API routes must not depend on theme**: All `/api/*` endpoints should work the same regardless of which theme served the page.
- **No trailing slash redirect**: Root path `/` is your default theme landing.

## Example: Gold Sniper HQ (production)

- `/` → Light theme from `static/index.html`
- `/?theme=pixel` → Pixel art theme from `DASHBOARD_HTML` constant
- Single file, port 8080
- API shared: `/api/overview`, `/api/agents`, `/api/chart`
