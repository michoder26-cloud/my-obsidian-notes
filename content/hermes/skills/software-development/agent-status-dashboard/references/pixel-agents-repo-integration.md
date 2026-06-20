# Pixel Agents Repo Integration Reference

## Source Repo
https://github.com/pixel-agents-hq/pixel-agents — 8.2k ⭐, MIT license

## Approach Comparison

| | Canvas + Sprites (Manual) | Embedded Vite Build (Preferred) |
|---|---|---|
| Fidelity | Approximate — must reimplement office layout, pathfinding | Exact — same code as original repo |
| Effort | Medium (write Canvas renderer + pathfinding) | Low (clone → npm install → vite build → iframe) |
| Risk | User says "มันไม่เหมือนอ่ะ" (doesn't look like it) | Matches exactly |
| Customizability | High (change colors, layout, animation) | Low (you get what the repo provides) |
| When to use | User wants a simplified/custom version | User wants the real pixel agents experience |

**Rule:** If the user references the GitHub repo or says it doesn't look right, switch to Approach B immediately. Do not try to fix the Canvas version piece by piece.

## Build + Embed Workflow (Approach B)

### Step 1: Clone
```bash
git clone --depth 1 https://github.com/pixel-agents-hq/pixel-agents.git /tmp/pixel-repo
```

### Step 2: Install dependencies
```bash
cd /tmp/pixel-repo/webview-ui
npm install
# ~60 packages, takes 10-30s
```

### Step 2.5: Modify source files for standalone production mode (CRITICAL)

Before building, you MUST modify three files to enable the browser mock in production builds:

**File 1:** `webview-ui/src/main.tsx` - remove `import.meta.env.DEV` guard:
```typescript
// BEFORE (line ~12):
  if (isBrowserRuntime && import.meta.env.DEV) {
    const { initBrowserMock } = await import('./browserMock.js');
    await initBrowserMock();
  }
// AFTER:
  if (isBrowserRuntime) {
    const { initBrowserMock } = await import('./browserMock.js');
    await initBrowserMock();
  }
```

**File 2:** `webview-ui/src/App.tsx` - remove the same guard:
```typescript
// BEFORE (line ~45):
    if (isBrowserRuntime && import.meta.env.DEV) {
      void import('./browserMock.js').then(({ dispatchMockMessages }) => dispatchMockMessages());
    }
// AFTER:
    if (isBrowserRuntime) {
      void import('./browserMock.js').then(({ dispatchMockMessages }) => dispatchMockMessages());
    }
```

**File 3:** `webview-ui/src/browserMock.ts` - change `shouldTryDecoded` to always try decoded JSON first:
```typescript
// BEFORE (line ~191):
  const shouldTryDecoded = import.meta.env.DEV;
// AFTER:
  const shouldTryDecoded = true;
```

Without these changes, the production build will show a blank/loading page because `initBrowserMock()` never runs.

### Step 3: Build (Vite)
```bash
npx vite build
# Output: ../dist/webview/
# Files: index.html + assets/index-xxx.js + assets/index-xxx.css
```

Build output structure:
```
dist/webview/
├── index.html                      # Entry point (React SPA)
├── assets/
│   ├── index-xxx.js                # ~290KB bundled JS
│   └── index-xxx.css               # ~20KB CSS
├── fonts/
├── characters.png
├── banner.png
└── Screenshot.jpg
```

### Step 4: Copy to static directory
```bash
# Build output
cp -r /tmp/pixel-repo/dist/webview /your-project/static/pixel-agents-build/

# Sprite assets (needed at runtime by browser mock)
cp -r /tmp/pixel-repo/webview-ui/public/assets /your-project/static/pixel-agents-build/assets/
```

### Step 5: Modify Python HTTP server to serve the build path

In `do_GET`, before the generic HTML route:
```python
if path.startswith("/assets/") or path.startswith("/pixel-agents-build/"):
    return self._serve_static()
```

The `_serve_static` method strips leading `/` and joins with `static/` dir. **Must handle SPA routing** — directory paths like `/pixel-agents-build/` need to serve `index.html`:

```python
def _serve_static(self):
    rel = self.path.lstrip("/")
    fpath = os.path.join(os.path.dirname(__file__), "static", rel)
    # CRITICAL: Serve index.html for directory paths (SPA routing)
    if os.path.isdir(fpath):
        fpath = os.path.join(fpath, "index.html")
    if not os.path.isfile(fpath):
        self.send_error(404)
        return
    # ... serve file with correct MIME type
```

Without this check, `/pixel-agents-build/` returns 404 because `os.path.isfile()` returns False for directories.

### Step 6: Embed in HTML
```html
<div id="hqCanvasContainer" style="background:#0f0f23;min-height:500px;">
  <iframe src="/pixel-agents-build/index.html"
    style="width:100%;height:520px;border:none;display:block;background:#0f0f23"></iframe>
</div>
```

### How Browser Mode Detection Works

The app auto-detects its runtime in `runtime.ts`:
```typescript
const runtime = typeof acquireVsCodeApi !== 'undefined' ? 'vscode' : 'browser';
```

In browser mode, `browserMock.ts` kicks in:
1. Fetches sprite PNGs via HTTP (`fetch('/assets/characters/char_0.png')`)
2. Decodes them at runtime into pixel data
3. Sends mock postMessage events (same format VS Code extension would send)
4. Office state, agents, and layout initialize from built-in `/assets/default-layout-1.json`

## Asset Layout

### Characters (`assets/characters/char_N.png`)
| File | Size | Frames | Directions |
|------|------|--------|------------|
| `char_0.png` — `char_5.png` | 112×96 RGBA | 7 per row (walk cycle) | 3 rows: down, up, right |

Frame dimensions: **16×32 px** per character.
Sprite sheet: 7 cols × 3 rows = 112 wide × 96 tall.

### Furniture (`assets/furniture/<NAME>/`)
Each furniture item has its own directory with PNG sprite(s) + `manifest.json`:

| Item | Key File | Size (px) | Tiles |
|------|----------|-----------|-------|
| Desk | `DESK_FRONT.png` | 48×32 | 3w × 2h |
| PC | `PC_FRONT_ON_1.png` | 16×32 | 1w × 2h |
| Wooden Chair | `WOODEN_CHAIR_FRONT.png` | 16×32 | 1w × 2h |
| Large Plant | `LARGE_PLANT.png` | 32×48 | 2w × 3h |
| Sofa | `SOFA_FRONT.png` | 32×16 | 2w × 1h |

Full manifest structure: Each furniture dir has `manifest.json` with:
- `width` / `height` in tiles (each tile = 16px)
- `states`: rotation groups, state groups (on/off for PC, etc.)
- `animationFrames`: optional animation data

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| Iframe shows blank white page | Build output not served correctly | Check `curl /pixel-agents-build/index.html` returns HTML, not 404 |
| Iframe shows dashboard page instead | Path not routed to `_serve_static()` | Add `path.startswith("/pixel-agents-build/")` guard before generic route |
| Sprites not loading | Browser mock can't fetch PNGs | Copy `public/assets/` dir to build output directory |
| Page stuck loading | JS error in iframe (CORS, missing deps) | Open iframe URL directly in browser and check DevTools console |
| "Could not resolve 'pngjs'" warning | npm pngjs import in core module | Warning only — Vite treats as external dep, browser mock handles PNG decode with native canvas API