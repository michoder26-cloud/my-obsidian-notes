---
name: pixel-agents-standalone
description: "Build and embed pixel-agents-hq/pixel-agents as a standalone web app (no VS Code) inside an existing dashboard server"
tags: [pixel-art, webview, embedding, React, Vite]
---

# Pixel Agents — Standalone Embed

## When to use

When the user asks to embed the [pixel-agents-hq/pixel-agents](https://github.com/pixel-agents-hq/pixel-agents) office into an existing web dashboard — not as a VS Code extension, but as a standalone web app served from any HTTP server.

## Quick Start — Minimal Steps

For an existing Python HTTP dashboard server on port 8080:

```bash
# 1. Clone & build (from pixel-agents repo root)
git clone --depth 1 https://github.com/pixel-agents-hq/pixel-agents.git
cd pixel-agents/webview-ui
npm install
# apply the 4 patches below, then:
npm run build

# 2. Copy to dashboard static dir (ADAPT THIS PATH)
mkdir -p /root/Claw_Trade/static/pixel-agents-build
cp -r ../dist/webview/* /root/Claw_Trade/static/pixel-agents-build/

# 3. Add route in Python handler — serve index.html at /agent-hq/
#    Must be added BEFORE any catch-all route that would shadow it.

# 4. Restart server & verify: curl http://localhost:8080/agent-hq/
#    Should contain vite-built JS bundles, NOT equityChart canvases.
```

## Prerequisites

- Node.js 20+ with npm
- The target dashboard server (Python/Node/whatever)
- `git clone` access

## Workflow

### 1. Clone & Build

```bash
git clone --depth 1 https://github.com/pixel-agents-hq/pixel-agents.git
cd pixel-agents/webview-ui
npm install
```

### 2. Source Modifications (CRITICAL)

The webview-ui is designed for VS Code's webview API + WebSocket standalone server. To run it standalone in a browser, make these **four** changes:

#### a) `src/main.tsx` — enable browser mock in production
Remove the `import.meta.env.DEV` guard so the browser mock always runs:
```
if (isBrowserRuntime) {  // was: if (isBrowserRuntime && import.meta.env.DEV)
```

#### b) `src/App.tsx` — dispatch mock messages in production
Same fix in the dispatch call:
```
if (isBrowserRuntime) {  // was: if (isBrowserRuntime && import.meta.env.DEV)
```

#### c) `src/browserMock.ts` — disable decoded JSON, use PNG fallback
Set `shouldTryDecoded` to `false` so it falls back to browser-side PNG decoding (the Vite dev-server middleware that serves decoded JSON does not exist in production):
```
const shouldTryDecoded = false;  // was: const shouldTryDecoded = import.meta.env.DEV;
```

#### d) `src/transport/index.ts` — mock postMessage transport
Replace WebSocket transport (which would reconnect forever) with a mock that listens to `window.postMessage` events from the browser mock:
```typescript
// Replace the entire createTransport() browser branch:
if (!isBrowserRuntime) {
  return new PostMessageTransport();
}
// Browser mock mode: listen to window.postMessage from browserMock
return {
  send: () => {}, // no-op
  onMessage: (handler) => {
    const listener = (e: MessageEvent) => {
      if (e.data && typeof e.data === 'object' && 'type' in e.data) {
        handler(e.data);
      }
    };
    window.addEventListener('message', listener);
    return () => window.removeEventListener('message', listener);
  },
  dispose: () => {},
};
```
Also remove the `WebSocketTransport` import line.

### 3. Build

```bash
npm run build
```

Build output goes to `../dist/webview/` by default (configured in `vite.config.ts`).

### 4. Deploy to Dashboard Server

Copy the built files to the dashboard's static directory:

```bash
mkdir -p /path/to/dashboard/static/pixel-agents-build
cp -r ../dist/webview/* /path/to/dashboard/static/pixel-agents-build/
```

### 5. Server-Side SPA Routing

The server MUST serve `index.html` when the directory path is requested. For a Python HTTP server (BaseHTTPRequestHandler), add this in the static file handler:

```python
if os.path.isdir(fpath):
    fpath = os.path.join(fpath, "index.html")
```

### 6. Embed via Iframe

In the dashboard HTML, add an iframe pointing to the build path:

```html
<iframe src="/pixel-agents-build/" style="width:100%;border:none;min-height:calc(100vh-120px)"></iframe>
```

## Architecture

```
Dashboard (port 8080)
  ├── /                         → Dashboard light theme
  ├── /api/*                    → Dashboard API
  └── /pixel-agents-build/      → Standalone pixel-agents (iframed)
                              ├── index.html
                              ├── assets/
                              │   ├── index-*.js       (React app)
                              │   ├── index-*.css
                              │   ├── furniture/
                              │   ├── floors/
                              │   ├── walls/
                              │   ├── characters/
                              │   └── furniture-catalog.json
                              ├── fonts/
                              └── ...
```

## How It Works

1. The production build (Vite) creates a SPA with `base: './'`
2. `main.tsx` imports `browserMock.ts` which loads assets (decoded JSON → PNG fallback)
3. `browserMock.ts` dispatches messages via `window.postMessage`
4. The mock transport in `transport/index.ts` listens to those same `window.postMessage` events
5. The React app receives character sprites, furniture, floor/wall tiles, and a default layout
6. The office renders on a Canvas element with Z-sorted entities

## References

- `references/p5js-sprite-integration.md` — Lightweight sprite injection into existing p5.js scenes (no React/Vite build needed). Covers sprite sheet anatomy, `drawingContext.drawImage()` workaround, variable shadowing trap table, and the try/catch debugging recipe.
- `references/agent-activity-state-machine.md` — CPU-driven agent behavior: agents walk between workstation and lounge based on Hermes API CPU data. Full state machine (resting → to_desk → working → to_rest), thresholds, movement, and lounge layout.
- `references/session-pitfalls-pixel-office-integration.md` — Session-specific pitfalls: backup strategy, delegation requirements, user preferences.

## Pitfalls

- **ALWAYS CHECK EXISTING INFRASTRUCTURE FIRST** — Critical. Before cloning/building/deploying from any YouTube video or external repo, verify whether the user already has a working Pixel Agent Office locally. Common locations to check: `~/pixel-agent-office/`, `~/pixel-agents/`, `~/pixel_hq/`, or any directory matching `*pixel*agent*`. Check for: (1) `server.py` with an `/api/status` endpoint, (2) running process on port 9120, (3) `index.html` with p5.js canvas and agent status fetch logic. If found, ask the user whether they want to ENHANCE the existing office (e.g., swap in better sprites) or REPLACE it entirely. Do NOT assume the user wants a fresh deployment from the video.
- **Server route ordering**: The `/agent-hq/` route must be checked **before** any catch-all dashboard route. If the server handler checks `/` first and serves the dashboard HTML, `/agent-hq/` will never be reached. Place the agent-hq route handler above the catch-all.
- **ALWAYS CHECK EXISTING INFRASTRUCTURE FIRST** — Critical. Before cloning/building/deploying from any YouTube video or external repo, verify whether the user already has a working Pixel Agent Office locally. Common locations to check: `~/pixel-agent-office/`, `~/pixel-agents/`, `~/pixel_hq/`, or any directory matching `*pixel*agent*`. Check for: (1) `server.py` with an `/api/status` endpoint, (2) running process on port 9120, (3) `index.html` with p5.js canvas and agent status fetch logic. If found, ask the user whether they want to ENHANCE the existing office (e.g., swap in better sprites) or REPLACE it entirely. Do NOT assume the user wants a fresh deployment from the video.
- **Server route ordering**: The `/agent-hq/` route must be checked **before** any catch-all dashboard route. If the server handler checks `/` first and serves the dashboard HTML, `/agent-hq/` will never be reached. Place the agent-hq route handler above the catch-all.
- **Blank page / "Loading..." stuck**: Almost always the transport issue. The WebSocketTransport tries forever to reconnect and blocks layoutReady. Fix: replace with mock postMessage transport.
- **Import.meta.env.DEV guards**: The browser mock code is gated behind `import.meta.env.DEV` in THREE places (main.tsx, App.tsx, browserMock.ts). All must be removed/changed for production builds.
- **Decoded JSON not found**: The Vite dev middleware serves `/assets/decoded/*.json` which doesn't exist in production. `shouldTryDecoded = true` causes a 404, then falls back to browser-side PNG decoding (slower but works).
- **SPA routing**: If the server returns 404 for `/pixel-agents-build/` (as a directory), the app won't load. Add directory → index.html logic in the static file handler.
- **Asset path resolution**: The built app uses relative paths (`./assets/...`). When served from `/pixel-agents-build/`, these resolve to `/pixel-agents-build/assets/...` — correct. Always verify the final URL resolution.

## Verification

1. Open `http://host:port/pixel-agents-build/` directly
2. Check browser console for:
   - `[BrowserMock] Ready (browser-png-decode)` — assets loaded
   - `[Webview] Received N pre-colored character sprites` — messages dispatched
   - No WebSocket reconnection errors
3. Confirm a `<canvas>` element exists in the DOM
4. Check the iframe loads in the dashboard tab