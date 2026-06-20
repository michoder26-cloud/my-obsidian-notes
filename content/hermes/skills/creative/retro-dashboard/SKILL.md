---
name: retro-dashboard
description: "Build retro video-game-style data dashboards with Python http.server backend + pixel-art CSS/JS frontend. CRT scan lines, Press Start 2P font, HP bars, sparkle particles, and real-time API data from SQLite."
platforms: [linux, macos, windows]
---

# Retro / Pixel Art Dashboard

Build a data dashboard that looks like a retro video game — CRT monitor aesthetic, pixel fonts, animated sparkle particles, HP bars, and bar charts styled as game UI elements.

## When to use

Use when the user asks for a:
- "game style" or "pixel" dashboard
- "BBR-style" or warm-toned retro office dashboard
- Retro-ui monitoring panel
- Fun / playful data display with real data
- Agent visualization with character sprites and HP/level stats
- "Pixel agents" walking around an office environment

## Architecture

```
┌──────────────────────────────────────────────────────┐
│  Python http.server (built-in, no deps)               │
│  ┌────────────────────────────────────────────────┐   │
│  │  GET /             → HTML page (retro theme)   │   │
│  │  GET /api/overview → JSON: stats + trades      │   │
│  │  GET /api/agents  → JSON: agent statuses       │   │
│  │  GET /api/chart   → JSON: equity curve         │   │
│  └────────────────────────────────────────────────┘   │
│  Reads from SQLite DB, learned_config.json, ps aux    │
└──────────────────────────────────────────────────────┘
```

## Color Palettes

### Dark Cyber Palette (Default — sci-fi green/blue)

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

### BBR Warm Palette (Retro Office / Golden-brown)

Use this variant when the user references a "BBR-style" dashboard, warm-toned pixel office, or the BrewBerich Co., Ltd. aesthetic. Extracted from image analysis of the reference design.

| Purpose | Hex | Usage |
|---------|-----|-------|
| Page BG | `#f5f0e0` | Light warm page background |
| Header BG | `#3d3224` | Dark brown navigation bar |
| Header text | `#e8d8a8` | Golden header links |
| Nav text | `#c8b888` | Navigation link text |
| Panel BG | `#fff8ee` | Card background (warm white) |
| Panel border | `#d8c8a0` | Card borders (tan) |
| Panel title | `#a37448` | Section heading accent |
| Body text | `#3d3224` | Dark brown body text |
| Dim text | `#7a6a4a` | Muted secondary labels |
| Floor tile A | `#8a7a4a` | Checkerboard tile (dark) |
| Floor tile B | `#9a8a5a` | Checkerboard tile (light) |
| Desk top | `#958a6a` | Wooden desk surface |
| Desk highlight | `#a89a78` | Desk highlight |
| CRT screen | `#38ba72` | Green monochrome monitor |
| Win green | `#56b846` | Positive P&L, active badges |
| Loss red | `#c0392b` | Negative P&L |
| Wall | `#17140f` | Office wall border |
| Divider | `#a37448` | Room divider lines |
| Agent body | `#e8d08a` | Pixel character body |
| Bubble BG | `rgba(255,255,240,0.95)` | Speech bubble background |
| Terminal header | `#46352d` | Status panel header |

#### BBR Agent Status Tags
```css
.tag.on  { background: #56b846; color: #fff; }    /* Working (green) */
.tag.off { background: #7a6a4a; color: #e8e0c8; } /* Idle (brown) */
```

#### BBR Dashboard Metric Cards
```css
.card { background: #fff8ee; border: 1px solid #d8c8a0; border-radius: 6px; padding: 16px; }
.card .num { font-size: 28px; font-weight: bold; margin-bottom: 2px; }
.card .lbl { font-size: 11px; color: #7a6a4a; }
```

## Key CSS Techniques

### Pixel Font
```css
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
font-family: 'Press Start 2P', monospace;
```

For BBR warm palette, use `'Courier New', monospace` for the terminal panel.

### CRT Scan Lines Overlay
```css
body::before {
  content: '';
  position: fixed; inset: 0;
  background: repeating-linear-gradient(
    0deg, transparent, transparent 2px,
    rgba(0,0,0,0.08) 2px, rgba(0,0,0,0.08) 4px
  );
  pointer-events: none;
  z-index: 9999;
}
```

### Retro Panel with Gold Accent
```css
.panel {
  background: #111128;
  border: 2px solid #2a2a5a;
  position: relative;
}
.panel::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #ffd700, transparent);
}
```

### BBR Panel Style (Warm)
```css
.panel {
  background: #fff8ee;
  border: 1px solid #d8c8a0;
  border-radius: 6px;
  padding: 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
}
.panel h3 {
  font-size: 13px; color: #a37448;
  margin-bottom: 10px;
  border-bottom: 1px solid #e8dcc0; padding-bottom: 6px;
}
```

### Animated Sparkle Particles
```css
@keyframes sparkle { 0% { opacity: 0; } 50% { opacity: 1; } 100% { opacity: 0; } }
.particle { animation: sparkle 3s infinite; }
```

### HP Bar
```css
.hp-bar { height: 6px; background: rgba(255,255,255,0.05); border-radius: 3px; overflow: hidden; }
.hp-fill { height: 100%; border-radius: 3px; transition: width 0.5s; }
```

### Live Status Dot
```css
.live-dot.on  { background: #00ff88; box-shadow: 0 0 8px #00ff88; animation: blink 1.5s infinite; }
.live-dot.off { background: #ff4757; }
```

## Python Backend Pattern

Use `http.server.HTTPServer` + custom `BaseHTTPRequestHandler`:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

PORT = 8080

class DashboardHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self._serve_html()
        elif self.path == "/api/overview":
            self._serve_json(self._get_overview())
        else:
            self.send_error(404)

    def _serve_html(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/html; charset=utf-8")
        self.send_header("Access-Control-Allow-Origin", "*")
        self.end_headers()
        self.wfile.write(DASHBOARD_HTML.encode("utf-8"))

    def _serve_json(self, data):
        self.send_response(200)
        self.send_header("Content-Type", "application/json")
        self.send_header("Access-Control-Allow-Origin", "*")
        self.end_headers()
        self.wfile.write(json.dumps(data, ensure_ascii=False).encode("utf-8"))

    def log_message(self, format, *args): pass  # Quiet

HTTPServer(("0.0.0.0", PORT), DashboardHandler).serve_forever()
```

## Data Sources

Common sources to wire up:
- **SQLite** — `sqlite3` with `row_factory = sqlite3.Row`
- **JSON files** — `learned_config.json`, trade logs
- **Running processes** — `subprocess.run(["ps", "aux"], ...)` to check bot status
- **Docker** — `subprocess.run(["docker", "ps", ...], ...)` for container status

## Canvas-Based Agent Animation (Walking Pixel Characters)

For a game-style experience where agents **walk around an office**, show activity speech bubbles, and animate in real-time — use an HTML `<canvas>` element with a `requestAnimationFrame` game loop instead of static CSS cards.

### Character FSM (Finite State Machine)

For agents that feel alive, use a **3-state FSM** matching the pixel-agents repo:

```
IDLE ──(wanderTimer)→ WALK ──(path complete)→ IDLE / TYPE
 │                                                    │
 └──(isActive=true & seat found)──────────────────────┘
```

| State | Behavior | Animation frames |
|-------|----------|-----------------|
| `IDLE` | Stands still, counts down `wanderTimer`. When timer fires: picks random walkable tile via BFS, transitions to WALK. After `wanderLimit` wanders, returns to seat. | Frame 0 (single idle pose) |
| `WALK` | Follows BFS path one tile at a time with smooth pixel interpolation. Updates direction each step. | Frames 0-3 (4-frame walk cycle) |
| `TYPE` | Sits at desk typing. When `isActive` becomes false, starts `seatTimer` countdown then transitions to IDLE. | Frames 4-5 (2-frame type animation) |

```javascript
function updateAgent(a, dt, walkableTiles) {
  a.frameTimer += dt;

  if (a.state === 'TYPE') {
    if (a.frameTimer >= 0.6) { a.frameTimer -= 0.6; a.frame = (a.frame + 1) % 2; }
    if (!a.isActive) {
      a.seatTimer -= dt;
      if (a.seatTimer <= 0) { a.state = 'IDLE'; a.wanderTimer = 1 + Math.random() * 2; }
    }
  }

  if (a.state === 'IDLE') {
    a.frame = 0; a.frameTimer = 0;
    if (a.isActive) { /* walk to seat */ return; }
    a.wanderTimer -= dt;
    if (a.wanderTimer <= 0) {
      a.wanderTimer = 1.5 + Math.random() * 3;
      if (a.wanderCount >= a.wanderLimit) {
        walkTo(a, a.seatCol, a.seatRow);
        a.wanderCount = 0;
      } else {
        var t = walkableTiles[Math.floor(Math.random() * walkableTiles.length)];
        walkTo(a, t.col, t.row);
        a.wanderCount++;
      }
    }
  }

  if (a.state === 'WALK') {
    if (a.frameTimer >= 0.15) { a.frameTimer -= 0.15; a.frame = (a.frame + 1) % 4; }
    // move toward next path node with smooth interpolation...
  }
}
```

Key constants for tuning:
- WANDER_PAUSE: 1.5–4.5 seconds (randomized per agent)
- WANDER_LIMIT: 3–6 wanders before returning to seat
- WALK_SPEED: 3–4 tiles/second
- TYPE_FRAME_DURATION: 0.5–0.7 seconds per frame

### Architecture

```
Canvas Game Loop (60fps)
├── drawOffice() — grid floor, walls, desks with monitors
├── updateAgents() — AI pathfinding: walk to desks, perform activities
│   ├── Each agent has {x, y, targetX, targetY, state, walkFrame}
│   ├── Activities: analyzing, reading, typing, deciding, learning, walking, idle
│   └── Random activity switching every 100-300 frames
├── drawAgent(a) — pixel character rendering
│   ├── Shadow ellipse → pixel body with legs → head → eyes → emoji badge → HP bar → level tag
│   └── Walking animation: leg offset alternates every 6 frames
└── Speech bubbles above agents when active (white bubble with name + role)
```

### Key Canvas Rendering

```javascript
const canvas = document.getElementById('agentCanvas');
const ctx = canvas.getContext('2d');

function drawAgent(a) {
  const { x, y, state: { emoji, color, activity, level, hp } } = a;
  
  // Shadow
  ctx.fillStyle = "rgba(0,0,0,0.3)";
  ctx.beginPath(); ctx.ellipse(x, y + 20, 14, 4, 0, 0, Math.PI * 2); ctx.fill();

  // Legs (walking animation: 4-frame cycle)
  const legOff = a.walkFrame < 2 ? 0 : (a.walkFrame % 2 === 0 ? 2 : -2);
  ctx.fillStyle = color;
  ctx.fillRect(x - 6, y + 2, 4, 6 + legOff);   // left leg
  ctx.fillRect(x + 3, y + 2, 4, 6 - legOff);   // right leg

  // Body (16x16 pixel block)
  ctx.fillStyle = color + "cc";
  ctx.fillRect(x - 8, y - 12, 16, 16);
  ctx.fillStyle = color;
  ctx.fillRect(x - 7, y - 12, 14, 15);

  // Head
  ctx.fillStyle = "#222"; ctx.fillRect(x - 8, y - 18, 16, 8);
  ctx.fillStyle = "#333"; ctx.fillRect(x - 7, y - 17, 14, 6);

  // Eyes (wider when typing/narrow when idle)
  const eyeW = activity === "typing" ? 2 : 3;
  ctx.fillStyle = activity === "deciding" ? "#ffd700" : "#fff";
  ctx.fillRect(x - 4, y - 15, eyeW, 2);
  ctx.fillRect(x + 1, y - 15, eyeW, 2);

  // Emoji above head
  ctx.font = "16px sans-serif"; ctx.textAlign = "center";
  ctx.fillText(emoji, x, y - 22);

  // Activity icon
  const labels = { analyzing: "🔮", reading: "📖", typing: "⌨️", deciding: "⚖️", learning: "🧠", walking: "🚶" };
  if (activity !== "idle") {
    ctx.font = "11px sans-serif";
    ctx.fillText(labels[activity] || "⚪", x - 14, y - 36);
  }

  // HP bar
  ctx.fillStyle = "#333"; ctx.fillRect(x - 10, y + 11, 20, 3);
  ctx.fillStyle = color;  ctx.fillRect(x - 10, y + 11, 20 * (hp / 100), 3);

  // Level badge
  ctx.fillStyle = "#222"; ctx.fillRect(x - 18, y + 6, 8, 6);
  ctx.fillStyle = "#ffd700"; ctx.font = "5px monospace"; ctx.textAlign = "center";
  ctx.fillText("Lv" + level, x - 14, y + 11);
}
```

### Grid-Based BFS Pathfinding (from sprite POV)

For pixel-accurate agent movement in a tile-based office where furniture blocks certain tiles, use **breadth-first search (BFS)** on a 2D grid instead of linear interpolation toward a target.

```javascript
const COLS = 24;  // Office grid width in tiles
const ROWS = 16;  // Office grid height in tiles
const TILE = 16;  // Pixels per tile (sprites are 16×16)

// Office grid: '.' = walkable, '#' = blocked (furniture/wall)
var officeGrid = [];

function buildGrid() {
  for (var y = 0; y < ROWS; y++) {
    officeGrid[y] = [];
    for (var x = 0; x < COLS; x++) {
      officeGrid[y][x] = '.';  // default walkable
    }
  }
  // Walls (border)
  for (var x = 0; x < COLS; x++) { officeGrid[0][x] = '#'; officeGrid[ROWS-1][x] = '#'; }
  for (var y = 0; y < ROWS; y++) { officeGrid[y][0] = '#'; officeGrid[y][COLS-1] = '#'; }
  // Block desk tiles (3 wide × 2 tall)
  var desks = [{x:3,y:5},{x:7,y:5},{x:11,y:5},{x:15,y:5}];
  desks.forEach(function(d) {
    for (var ty = 0; ty < 2; ty++)
      for (var tx = 0; tx < 3; tx++)
        if (d.y+ty < ROWS && d.x+tx < COLS) officeGrid[d.y+ty][d.x+tx] = '#';
  });
}

function findPath(sx, sy, ex, ey) {
  if (sx === ex && sy === ey) return [sx, sy];
  var queue = [[sx, sy]];
  var visited = {};
  var parent = {};
  var key = function(x, y) { return x + ',' + y; };
  visited[key(sx, sy)] = true;

  while (queue.length > 0) {
    var cur = queue.shift();
    var cx = cur[0], cy = cur[1];
    var neighbors = [[cx+1,cy], [cx-1,cy], [cx,cy+1], [cx,cy-1]];
    for (var n = 0; n < 4; n++) {
      var nx = neighbors[n][0], ny = neighbors[n][1];
      var k = key(nx, ny);
      if (visited[k]) continue;
      if (nx < 0 || nx >= COLS || ny < 0 || ny >= ROWS) continue;
      if (officeGrid[ny][nx] === '#') continue;  // blocked

      visited[k] = true;
      parent[k] = [cx, cy];
      if (nx === ex && ny === ey) {
        // Reconstruct path
        var path = [[nx, ny]];
        var curK = k;
        while (parent[curK]) {
          var p = parent[curK];
          path.unshift(p);
          curK = key(p[0], p[1]);
        }
        return path;
      }
      queue.push([nx, ny]);
    }
  }
  return null;  // No path found
}

// Agent class with BFS movement
function Agent(idx, name, title, color) {
  this.idx = idx;
  this.name = name;
  this.color = color;
  this.x = 3 + (idx % 6) * 3;  // Starting tile position
  this.y = 3 + Math.floor(idx / 3) * 3;
  this.px = this.x * TILE;     // Pixel position (smooth interpolation)
  this.py = this.y * TILE;
  this.targetX = this.x;
  this.targetY = this.y;
  this.speed = 24;             // Pixels per second
  this.walking = false;
  this.path = [];
  this.frame = 0;
  this.frameTimer = 0;
  this.dir = 0;                // 0=down, 1=left, 2=right, 3=up
}

Agent.prototype.walkTo = function(tx, ty) {
  var path = findPath(Math.round(this.x), Math.round(this.y), tx, ty);
  if (path && path.length > 1) {
    this.path = path.slice(1);
    this.targetX = path[path.length - 1][0];
    this.targetY = path[path.length - 1][1];
    this.walking = true;
  }
};

Agent.prototype.update = function(dt) {
  if (this.walking && this.path.length > 0) {
    var nx = this.path[0][0], ny = this.path[0][1];
    var targetPx = nx * TILE, targetPy = ny * TILE;
    var dx = targetPx - this.px, dy = targetPy - this.py;
    var dist = Math.sqrt(dx * dx + dy * dy);
    var step = this.speed * dt;

    if (step >= dist) {
      this.px = targetPx;
      this.py = targetPy;
      this.x = nx;
      this.y = ny;
      this.path.shift();
      // Set direction based on movement
      this.dir = Math.abs(dx) > Math.abs(dy) ? (dx > 0 ? 2 : 1) : (dy > 0 ? 0 : 3);
      if (this.path.length === 0) this.walking = false;
    } else {
      this.px += (dx / dist) * step;
      this.py += (dy / dist) * step;
      this.dir = Math.abs(dx) > Math.abs(dy) ? (dx > 0 ? 2 : 1) : (dy > 0 ? 0 : 3);
    }
  }
  // Walking animation: cycle frames 0-3
  if (this.walking) {
    this.frameTimer += dt;
    if (this.frameTimer >= 0.15) { this.frame = (this.frame + 1) % 4; this.frameTimer = 0; }
  } else {
    this.frame = 0;
    this.frameTimer = 0;
  }
};
```

### Office Environment — Dark Cyber Style
```javascript
function drawOffice() {
  // Floor
  ctx.fillStyle = "#1a1a3e"; ctx.fillRect(0, 0, W, H);
  // Grid
  ctx.strokeStyle = "rgba(42,42,90,0.3)"; ctx.lineWidth = 1;
  for (let x = 0; x < W; x += 32) ctx.strokeRect(x, 0, 0, H);
  for (let y = 0; y < H; y += 32) ctx.strokeRect(0, y, W, 0);
  // Walls
  ctx.fillStyle = "#2a2a5a"; ctx.fillRect(0, 0, W, 8); ctx.fillRect(0, 0, 8, H);
  ctx.fillRect(W - 8, 0, 8, H); ctx.fillRect(0, H - 8, W, 8);
  // Room dividers
  for (let i = 1; i < 3; i++) ctx.fillStyle = "#2a2a5a", ctx.fillRect(i * 380 - 3, 40, 6, H - 80);
  // Desks with monitors
  DESKS.forEach((d, i) => {
    ctx.fillStyle = "#3a3a6a"; ctx.fillRect(d.x - 28, d.y - 12, 56, 24);
    ctx.fillStyle = "#4a4a7a"; ctx.fillRect(d.x - 24, d.y - 8, 48, 16);
    ctx.fillStyle = "#111"; ctx.fillRect(d.x - 10, d.y - 24, 20, 16);
    ctx.fillStyle = "#2a2a5a"; ctx.fillRect(d.x - 8, d.y - 22, 16, 12);
    if (agents[i].state.activity !== "idle") {
      ctx.fillStyle = agents[i].state.color + "30";
      ctx.fillRect(d.x - 7, d.y - 21, 14, 10); // screen glow
    }
    ctx.fillStyle = "#576574"; ctx.font = "9px monospace"; ctx.textAlign = "center";
    ctx.fillText(AGENTS[i].name, d.x, d.y + 28);
  });
}
```

### Office Environment — BBR Warm Style

For the BBR warm palette, the office uses wooden desks, CRT green monitors, brown floor tiles, and potted plants:

```javascript
const W = 1140, H = 800;  // Canvas dimensions
const CELL = 48;
const DESKS = [
  {x:1,y:1,w:3,h:2,i:0},{x:1,y:4,w:3,h:2,i:1},
  {x:7,y:1,w:3,h:2,i:2},{x:7,y:4,w:3,h:2,i:3},
  {x:13,y:1,w:3,h:2,i:4},{x:13,y:4,w:3,h:2,i:5},
];

function drawOffice() {
  // Floor tiles — warm brown checkerboard
  for (var y = 0; y < 6; y++) for (var x = 0; x < 5; x++) {
    var shade = (x + y) % 2 === 0 ? "#8a7a4a" : "#9a8a5a";
    ctx.fillStyle = shade;
    ctx.fillRect(x * CELL * 3, y * CELL * 2, CELL * 3, CELL * 2);
    ctx.strokeStyle = "rgba(0,0,0,0.1)"; ctx.lineWidth = 1;
    ctx.strokeRect(x * CELL * 3, y * CELL * 2, CELL * 3, CELL * 2);
  }
  // Walls
  ctx.fillStyle = "#17140f"; ctx.fillRect(0, 0, W, 6); ctx.fillRect(0, 0, 6, H);
  ctx.fillRect(W - 6, 0, 6, H); ctx.fillRect(0, H - 6, W, 6);
  // Divider lines at column 5.5 and 11
  ctx.fillStyle = "#a37448"; ctx.fillRect(CELL * 5.5 - 2, 8, 4, H - 16);
  ctx.fillRect(CELL * 11 - 2, 8, 4, H - 16);

  DESKS.forEach(function(d) {
    var cx = d.x * CELL, cy = d.y * CELL, cw = d.w * CELL, ch = d.h * CELL;
    
    // Desk shadow
    ctx.fillStyle = "rgba(0,0,0,0.2)"; ctx.fillRect(cx + 3, cy + 3, cw, ch);
    // Desk top (wood color)
    ctx.fillStyle = "#958a6a"; ctx.fillRect(cx, cy, cw, ch);
    ctx.fillStyle = "#a89a78"; ctx.fillRect(cx + 2, cy + 2, cw - 4, ch - 4);
    // Wood grain lines
    for (var g = 0; g < ch; g += 4) {
      ctx.fillStyle = "rgba(0,0,0,0.04)";
      ctx.fillRect(cx + 2, cy + g, cw - 4, 1);
    }
    // CRT Monitor — green screen
    var mx = cx + 12, my = cy - 6;
    ctx.fillStyle = "#222"; ctx.fillRect(mx, my, 24, 18);
    ctx.fillStyle = "#333"; ctx.fillRect(mx + 1, my + 1, 22, 16);
    ctx.fillStyle = agents[d.i].act !== "idle" ? agents[d.i].c + "40" : "#1a2a1a";
    ctx.fillRect(mx + 2, my + 2, 20, 14);
    if (agents[d.i].act !== "idle") {
      ctx.fillStyle = agents[d.i].c + "80";
      ctx.fillRect(mx + 4, my + 4, 16, 2);
      ctx.fillRect(mx + 4, my + 8, 10, 2);
    }
    // Keyboard
    ctx.fillStyle = "#6a5a3a"; ctx.fillRect(cx + 4, cy + ch - 4, cw - 8, 3);
    for (var k = 0; k < 8; k++) {
      ctx.fillStyle = "#8a7a5a"; ctx.fillRect(cx + 6 + k * 4, cy + ch - 3, 3, 2);
    }
    // Chair
    var chx = cx + cw / 2 - 7, chy = cy + ch + 6;
    ctx.fillStyle = "#8a7a5a"; ctx.fillRect(chx, chy, 14, 14);
    ctx.fillStyle = "#a89a78"; ctx.fillRect(chx + 1, chy + 1, 12, 12);
    ctx.fillStyle = "#6a5a3a"; ctx.fillRect(chx - 1, chy - 4, 16, 4); // chair back
    // Name label
    ctx.fillStyle = "#a09070"; ctx.font = "8px monospace"; ctx.textAlign = "center";
    ctx.fillText(TEAM[d.i].n || "", cx + cw / 2, cy + ch + 26);
    // Potted plant (corners)
    if (d.i === 0 || d.i === 5) {
      var px = d.i === 0 ? cx - 16 : cx + cw + 2;
      ctx.fillStyle = "#3a2a1a"; ctx.fillRect(px, cy + ch - 8, 10, 8);
      ctx.fillStyle = "#4a7a3a";
      ctx.beginPath(); ctx.arc(px + 5, cy + ch - 14, 9, 0, Math.PI * 2); ctx.fill();
      ctx.fillStyle = "#5a9a4a";
      ctx.beginPath(); ctx.arc(px + 5, cy + ch - 18, 6, 0, Math.PI * 2); ctx.fill();
    }
  });
}
```

### Critical: Z-Sorted Rendering (Full Depth Ordering)

**⚠️ DO NOT just sort agents by Y.** The correct approach (matching the pixel-agents-hq game engine exactly) is to collect ALL drawable objects — furniture AND characters — into a single array with explicit Z values, sort by Z ascending, then draw. This ensures proper depth ordering when a character sits at a desk or walks in front of/behind furniture.

#### Z-Value Assignment Rules

| Object | Z Value | Rationale |
|--------|---------|-----------|
| PC monitor | `(desk.y - 2) * TILE` | Highest on screen = behind everything |
| Plant | `1 * TILE` | Background decor |
| Character **standing** | `(row + 1) * TILE` | Feet position on grid |
| Desk | `(desk.y + 2) * TILE + 0.1` | Sits on top of character's lower body |
| Chair | `(desk.y + 3) * TILE` | Lowest on screen = in front of everything |

The +0.1 offset on desks is critical — it ensures the desk sorts AFTER the character standing at the same row, so the desk covers the character's lower body (creating the sitting illusion).

#### Sitting Offset — The Missing Piece

The +0.1 Z-offset alone is NOT enough. You also need a **vertical sitting offset** (shifts the character's sprite down) so the desk covers more of their body:

```
SITTING_OFFSET = 12px   # at 3x scale (4px at native 16px scale)
```

Without this, a character at row 3 (top of desk rows 3-4) has their sprite covering Y=106 to Y=202, while the desk covers Y=154 to Y=250. Only the bottom 48px overlap → the character looks like they're STANDING behind the desk.

With SITTING_OFFSET=12:
- Character sprite shifts to Y=118 to Y=214
- Desk covers Y=154 to Y=214 (60px overlap)
- Character visible above desk: 118-154 = 36px (just head and shoulders)
- **Looks like SITTING at the desk**

```javascript
// In the character draw section:
const sittingOffset = a.sitting ? 12 : 0;  // 12px at 3x scale
const sy = py + sittingOffset;
ctx.drawImage(img, fCol*FW, dirRow*FH, FW, FH, px, sy, FW*SC, FH*SC);
ctx.fillText(a.name, px+FW*SC/2, sy-4);
```

#### Typing Animation While Sitting

When agents are at their desks and active, animate a subtle typing loop:

```javascript
// Update loop — typing animation (slow frame cycle)
if (a.sitting) {
  a.ft += 1/60;
  if (a.ft > 0.3) { a.frame = (a.frame + 1) % 4; a.ft = 0; }
}

// Frame selection — differentiate walking vs sitting
const fCol = a.walk ? a.frame % 7 : (a.sitting ? a.frame % 4 : 0);

// Green sparkle particles near keyboard area (sitting only)
if (a.sitting) {
  const spark = Math.sin(a.idleOff * 8) * 0.5 + 0.5;  // oscillate
  ctx.fillStyle = `rgba(0,255,136,${spark * 0.6})`;
  ctx.fillRect(px+FW*SC/2-2, sy+FH*SC-20, 4, 2);  // keyboard spark
  ctx.fillRect(px+FW*SC/2-1, sy+FH*SC-16, 2, 2);
}
```

#### Walk + Return-to-Desk Scheduler

Use TWO setIntervals: one for random wandering, one for returning to the assigned desk:

```javascript
// 1. Random wander every 3-6 seconds (30% chance)
setInterval(() => {
  if (!a.walk && Math.random() < 0.3) {
    const t = walkableTargets[Math.floor(Math.random() * walkableTargets.length)];
    const path = findPath(Math.round(a.px), Math.round(a.py), t[0], t[1]);
    if (path.length > 1) { a.path = path; a.walk = true; a.sitting = false; }
  }
}, 3000 + Math.random() * 3000);

// 2. Return to desk after wandering (fires regardless, checks if already seated)
setInterval(() => {
  if (!a.walk && !a.sitting) {
    const path = findPath(Math.round(a.px), Math.round(a.py), a.col, a.row);
    if (path.length > 1) { a.path = path; a.walk = true; }
  }
}, 5000 + Math.random() * 4000);
```

When the character's path completes and they arrive at their desk, set `sitting = true` and the sitting offset + desk Z-coverage makes them look seated.

#### Correct Rendering Code

```javascript
// 1. Floor (always first — no Z-sort needed)
for (let y=0; y<ROWS; y++) for (let x=0; x<COLS; x++) {
  /* draw floor tiles */
}

// 2. Collect ALL drawable objects with Z values
const drawList = [];

// PCs (high on screen = low Z)
desks.forEach(d => {
  const z = (d.y - 2) * TILE;
  const pcPx = /* calculate */, pcPy = /* calculate */;
  if (pc sprite loaded) {
    drawList.push({z, draw: () => ctx.drawImage(pc, pcPx, pcPy, w, h)});
  }
});

// Plants
drawList.push({z: 1 * TILE, draw: () => /* draw plant */});

// Characters (Z = feet position)
agents.forEach(a => {
  const z = a.py * TILE + TILE;
  const px = a.px * TILE + OX;
  const py = a.py * TILE + OY + TILE - FH * SC;  // sprite top = 1 tile above feet
  drawList.push({
    z,
    draw: () => {
      ctx.drawImage(/* sprite sheet frame */);
      ctx.fillText(a.name, /* name position */);
    }
  });
});

// Desks (slightly higher Z than characters at same level)
desks.forEach(d => {
  const z = (d.y + 2) * TILE + 0.1;  // +0.1 for deterministic sorting
  drawList.push({z, draw: () => ctx.drawImage(deskSprite, /* ... */)});
});

// Chairs (in front of everything at this desk row)
desks.forEach(d => {
  const z = (d.y + 3) * TILE;
  drawList.push({z, draw: () => ctx.drawImage(chairSprite, /* ... */)});
});

// 3. Sort by Z ascending and draw
drawList.sort((a, b) => a.z - b.z);
drawList.forEach(item => item.draw());
```

#### Game Loop
```javascript
function gameLoop() {
  updateAgents();
  ctx.clearRect(0, 0, W, H);
  drawOfficeFloor();   // Step 1
  drawZSorted();       // Steps 2-3 (all objects sorted by Z)
  drawTitleBar();      // UI overlay on top
  requestAnimationFrame(gameLoop);
}
gameLoop();
```

### Speech Bubbles (BBR Style)
```javascript
function drawBubble(a) {
  if (!a.bub) return;
  var bw = 56, bh = 18, bx = a.x - bw / 2, by = a.y - 32;
  ctx.fillStyle = "rgba(255,255,240,0.95)";
  ctx.fillRect(bx, by, bw, bh);
  ctx.strokeStyle = "#a09070"; ctx.lineWidth = 1; ctx.strokeRect(bx, by, bw, bh);
  // Tail triangle
  ctx.fillStyle = "rgba(255,255,240,0.95)";
  ctx.beginPath(); ctx.moveTo(a.x - 4, by + bh);
  ctx.lineTo(a.x + 4, by + bh); ctx.lineTo(a.x, by + bh + 5); ctx.closePath(); ctx.fill();
  // Text: name in bold, role underneath
  ctx.fillStyle = "#3d3224"; ctx.font = "bold 6px monospace"; ctx.textAlign = "center";
  ctx.fillText(a.n, a.x, by + 8);
  ctx.fillStyle = "#7a6a4a"; ctx.font = "5px monospace";
  ctx.fillText("· " + a.r, a.x, by + 15);
}
```

## Interactive Side Panel with Click-to-Walk

When the Agent HQ has a canvas office on the left and a side panel on the right, make the side panel **interactive** — clicking an agent in the list makes them walk to their desk on canvas.

### Layout

```html
<div id="agentHQ" style="display:flex;height:100%">
  <div id="canvasWrap" style="flex:1;position:relative;overflow:hidden">
    <canvas id="officeCanvas"></canvas>
  </div>
  <div id="sidePanel" style="width:280px;background:var(--panel);border-left:2px solid var(--border);overflow-y:auto;padding:10px">
    <div class="sp-title">👥 AGENT STATUS</div>
    <div id="agentList"></div>
    <!-- System info below agent list -->
    <div style="margin-top:10px;padding-top:8px;border-top:1px solid var(--border)">
      <div style="font-family:var(--pixel);font-size:7px;color:var(--dim);margin-bottom:6px">SYSTEM</div>
      <div style="font-size:10px;display:flex;justify-content:space-between">
        <span>Bot:</span><span id="sysBot" style="color:var(--green);font-family:var(--pixel);font-size:7px">--</span>
      </div>
    </div>
  </div>
</div>
```

### Agent List HTML (generated per agent)

```html
<div class="sp-agent" data-idx="0">
  <div class="sp-agent-top">
    <canvas class="sp-agent-preview" id="preview0"></canvas>
    <div class="sp-agent-info">
      <div class="sp-agent-name">Quant Analyst</div>
      <div class="sp-agent-title">เทคนิคัล วิซาร์ด</div>
      <span class="sp-agent-status active">🟢 WORKING</span>
    </div>
  </div>
  <div class="sp-hp"><div class="sp-hp-fill" style="width:85%"></div></div>
  <div class="sp-agent-duty">RSI, MACD, EMA, Fibo</div>
</div>
```

### CSS

```css
#agentHQ { display: flex; height: 100%; gap: 0; }
#canvasWrap { flex: 1; position: relative; background: #0a0a1a; overflow: hidden; }
#sidePanel { width: 280px; background: var(--panel); border-left: 2px solid var(--border); overflow-y: auto; flex-shrink: 0; padding: 10px; }
@media (max-width: 900px) { #sidePanel { width: 200px; } }
@media (max-width: 700px) { #agentHQ { flex-direction: column; } #sidePanel { width: 100%; max-height: 200px; } }

.sp-title { font-family: var(--pixel); font-size: 8px; color: var(--gold); margin-bottom: 10px; padding-bottom: 6px; border-bottom: 1px solid var(--border); }
.sp-agent { background: var(--panel-alt); border: 1px solid var(--border); border-radius: 6px; padding: 8px; margin-bottom: 6px; cursor: pointer; transition: all 0.2s; }
.sp-agent:hover { border-color: var(--gold); transform: translateX(-2px); }
.sp-agent.selected { border-color: var(--gold); box-shadow: 0 0 10px rgba(255,215,0,0.1); }
.sp-agent-top { display: flex; align-items: center; gap: 8px; }
.sp-agent-preview { width: 32px; height: 32px; image-rendering: pixelated; border-radius: 4px; border: 1px solid var(--border); flex-shrink: 0; }
.sp-agent-name { font-family: var(--pixel); font-size: 7px; color: var(--text); margin-bottom: 2px; }
.sp-agent-title { font-size: 9px; color: var(--dim); }
.sp-agent-status { font-size: 7px; font-family: var(--pixel); padding: 1px 6px; border-radius: 8px; display: inline-block; }
.sp-agent-status.active { background: rgba(0,255,136,0.15); color: var(--green); }
.sp-agent-status.idle { background: rgba(87,101,116,0.15); color: var(--dim); }
.sp-hp { margin-top: 6px; height: 4px; background: rgba(255,255,255,0.05); border-radius: 2px; overflow: hidden; }
.sp-hp-fill { height: 100%; border-radius: 2px; }
.sp-agent-duty { font-size: 9px; color: var(--dim); margin-top: 4px; padding-left: 40px; }
```

### Click-to-Walk Interaction

```javascript
// When an agent list item is clicked
document.querySelectorAll('.sp-agent').forEach(function(el) {
  el.addEventListener('click', function() {
    var idx = parseInt(el.dataset.idx);
    var agent = agents[idx];
    if (!agent) return;
    
    selectedAgent = agent;
    // Walk to their desk position (adjust per layout)
    var deskX = 3 + (idx % 4) * 4;
    var deskY = idx < 4 ? 8 : 13;
    agent.walkTo(deskX, deskY);
    agent.say('📍 ' + agent.name, 3);
    updateSidePanel();  // Re-render to update selected state
  });
});

// Click on canvas to walk or select
document.getElementById('canvasWrap').addEventListener('click', function(e) {
  var rect = canvas.getBoundingClientRect();
  var mx = e.clientX - rect.left;
  var my = e.clientY - rect.top;
  
  // Convert to tile coordinates
  var tileX = (mx - offsetX) / SCALE;
  var tileY = (my - offsetY) / SCALE;
  
  // Check if any agent was clicked (last drawn = topmost)
  var clicked = null;
  for (var i = agents.length - 1; i >= 0; i--) {
    var a = agents[i];
    var dx = tileX - a.px / TILE;
    var dy = tileY - a.py / TILE;
    if (dx > -0.5 && dx < 1.5 && dy > -0.5 && dy < 1.5) {
      clicked = a;
      break;
    }
  }
  
  if (clicked) {
    selectedAgent = clicked;
    clicked.say('✅ Report!', 3);
  } else if (selectedAgent && tileX > 0 && tileX < COLS-1 && tileY > 0 && tileY < ROWS-1) {
    selectedAgent.walkTo(Math.floor(tileX), Math.floor(tileY));
  }
});
```

### Preview Sprite in Side Panel

Draw a small version of the character's sprite from the spritesheet into the preview canvas:

```javascript
function drawPreview(idx) {
  var pCanvas = document.getElementById('preview' + idx);
  if (!pCanvas) return;
  var pc = pCanvas.getContext('2d');
  var img = charSheets[agents[idx].sheetIdx];  // or just idx % 6
  // Draw frame 4 (idle) of this character's row
  pc.clearRect(0, 0, 32, 32);
  if (img && img.complete && img.naturalWidth > 0) {
    // frame=4 (idle), row=character index
    pc.drawImage(img, 4 * 16, (idx % 6) * 16, 16, 16, 0, 0, 32, 32);
  } else {
    // Fallback colored square
    pc.fillStyle = agents[idx].color || '#fff';
    pc.fillRect(4, 4, 24, 24);
  }
}
```

## Terminal / Status Panel (BBR Right Side)

The right side of the BBR dashboard layout (38% width) features a warm-beige status panel with:
- Panel header in dark brown (`#46352d`) with golden text
- Agent counts (Active / Idle) displayed as cards
- Separated agent list by status group

### Layout
```html
<div class="split">
  <div class="left" style="flex:0 0 62%;"><canvas id="gc"></canvas></div>
  <div class="right" style="flex:0 0 38%;background:#c8b888;border-left:2px solid #a37448;">
    <div class="terminal">
      <div class="panel-header">📡 TEAM STATUS · สถานะทีม</div>
      <div class="sync">🔄 Sync: {time}</div>
      <hr>
      <div class="counts">
        <div class="count active"><div class="num">{activeCount}</div><div class="lbl">กำลังทำงาน</div></div>
        <div class="count"><div class="num">{idleCount}</div><div class="lbl">ว่าง / Idle</div></div>
      </div>
      <hr>
      <div class="grptitle">🔥 กำลังทำงาน</div>
      {active agents list with .tag.on}
      <div class="grptitle">💤 ว่าง</div>
      {idle agents list with .tag.off}
    </div>
  </div>
</div>
```

### CSS
```css
.terminal { padding: 16px; font-family: 'Courier New', monospace; color: #1a1a0a; }
.terminal .panel-header { background: #46352d; color: #e8d8a8; padding: 6px 12px; font-size: 12px; font-weight: bold; margin: -16px -16px 12px; letter-spacing: 1px; }
.terminal .sync { color: #7a6a4a; font-size: 10px; margin-bottom: 12px; }
.terminal hr { border: none; border-top: 1px solid #a09070; margin: 10px 0; }
.counts { display: flex; gap: 12px; margin-bottom: 12px; }
.count { text-align: center; padding: 6px 14px; border: 1px solid #a09070; border-radius: 3px; background: rgba(255,255,240,0.4); }
.count .num { font-size: 26px; font-weight: bold; color: #3d3224; }
.count .lbl { font-size: 9px; color: #7a6a4a; }
.count.active .num { color: #56b846; }
.grptitle { color: #3d3224; font-size: 11px; font-weight: bold; margin: 10px 0 4px 4px; }
.agent-row { display: flex; align-items: center; gap: 6px; padding: 4px 6px; font-size: 11px; border-radius: 2px; }
.agent-row:hover { background: rgba(255,255,240,0.4); }
.agent-row .name { font-weight: bold; flex: 1; }
.agent-row .role { color: #7a6a4a; font-size: 9px; }
.tag.on { background: #56b846; color: #fff; padding: 1px 5px; border-radius: 2px; font-size: 8px; }
.tag.off { background: #7a6a4a; color: #e8e0c8; padding: 1px 5px; border-radius: 2px; font-size: 8px; }
```

## Multi-Page SPA with Tab Navigation

When the user wants multiple pages (dashboard, agents, trades, settings), the best approach is a **single-page application (SPA)** — all pages in one HTML file, shown/hidden via JavaScript tabs. This avoids full-page reloads and keeps the canvas animation running when switching tabs.

### HTML Structure

```html
<div class="nav" id="navTabs">
  <button class="nav-btn active" data-page="dash">📊 Dashboard</button>
  <button class="nav-btn" data-page="hq">🏢 Agent HQ</button>
  <button class="nav-btn" data-page="trades">📋 Trades</button>
  <button class="nav-btn" data-page="settings">⚙️ Settings</button>
</div>

<div class="content">
  <div class="page active" id="page-dash"><!-- Dashboard content --></div>
  <div class="page" id="page-hq"><!-- Agent HQ with canvas --></div>
  <div class="page" id="page-trades"><!-- Trades table --></div>
  <div class="page" id="page-settings"><!-- Settings pane --></div>
</div>
```

### CSS

```css
.content { flex: 1; display: flex; overflow: hidden; }
.page { display: none; width: 100%; height: 100%; overflow-y: auto; padding: 12px; }
.page.active { display: block; }

.nav { display: flex; gap: 0; background: var(--panel-alt); border-bottom: 2px solid var(--border); flex-shrink: 0; }
.nav-btn {
  font-family: var(--pixel); font-size: 8px; padding: 10px 20px;
  background: transparent; border: none; color: var(--dim);
  cursor: pointer; transition: all 0.2s;
  border-bottom: 3px solid transparent; white-space: nowrap;
  display: flex; align-items: center; gap: 6px;
}
.nav-btn:hover { color: var(--text); background: rgba(255,255,255,0.03); }
.nav-btn.active { color: var(--gold); border-bottom-color: var(--gold); background: rgba(255,215,0,0.05); }
```

### JavaScript Tab Switching

```javascript
document.querySelectorAll('.nav-btn').forEach(function(btn) {
  btn.addEventListener('click', function() {
    document.querySelectorAll('.nav-btn').forEach(function(b) { b.classList.remove('active'); });
    document.querySelectorAll('.page').forEach(function(p) { p.classList.remove('active'); });
    btn.classList.add('active');
    document.getElementById('page-' + btn.dataset.page).classList.add('active');
    // Canvas needs resize when switching to agent HQ
    if (btn.dataset.page === 'hq' && window.resizeCanvas) {
      window.resizeCanvas();
    }
  });
});
```

### Canvas Lifecycle with Tab Switching

When the agent HQ page uses `requestAnimationFrame` for a game loop, the animation must keep running even when other tabs are active, otherwise agents freeze:

```javascript
var animId;

function gameLoop(time) {
  // Always keep looping — even if HQ tab is hidden the animation continues
  if (agents.length > 0) {
    var dt = lastTime ? Math.min((time - lastTime) / 1000, 0.1) : 0.016;
    lastTime = time;
    agents.forEach(function(a) { a.update(dt); });
    if (document.getElementById('page-hq').classList.contains('active')) {
      renderOffice();  // Only draw when visible
    }
  }
  animId = requestAnimationFrame(gameLoop);
}
```

### Python Backend for SPA

The Python server serves the same HTML for all routes (or redirects everything to `/`):

```python
class DashboardHandler(BaseHTTPRequestHandler):
    api = DashboardAPI()

    def do_GET(self):
        path = self.path.split("?")[0]

        # API routes
        if path == "/api/overview":
            return self._serve_json(self.api.get_overview())
        elif path == "/api/agents":
            return self._serve_json(self.api.get_agents_status())
        elif path == "/api/chart":
            return self._serve_json(self.api.get_performance_chart())

        # Static assets (e.g. sprite PNGs)
        if path.startswith("/assets/"):
            return self._serve_static()

        # All other routes → SPA (same HTML for every page)
        self._serve_html()
```

## Multi-Page Architecture (Alternative — Separate HTML Pages)

For deployments where SEO, deep linking, or very different page structures matter, serve different HTML per route (but note: this kills canvas animations on page navigation).

```python
# In your BaseHTTPRequestHandler:
NAV_BAR = """<nav>
  <a href="/">Dashboard</a>
  <a href="/agents">Agent HQ</a>
  <a href="/trades">Trades</a>
  <a href="/settings">Settings</a>
  <span id="liveDot"></span>
</nav>"""

PAGES = {}  # Cache built HTML

def do_GET(self):
    path = urllib.parse.urlparse(self.path).path
    if path.startswith("/api/"):
        self._serve_json(api_endpoints[path]())
    elif path == "/":
        self._serve_html(PAGES.get("dashboard"))
    elif path in PAGES:
        self._serve_html(PAGES[path])
    else:
        self.send_error(404)

# Build pages at startup
PAGES["dashboard"] = build_dashboard(None)  # No nav highlight
PAGES["/agents"] = build_agents()
# etc.
```

Each page is generated by a function that returns a complete HTML string. The `NAV_BAR` is injected into every page for consistent navigation, and the live status dot updates via a shared JS snippet.

For the BBR warm palette, the nav bar uses a dark brown header:
```css
.header { background: #3d3224; border-bottom: 2px solid #a37448; display: flex; align-items: center; }
.header a { color: #c8b888; font-size: 12px; padding: 4px 12px; }
.header a:hover { color: #e8d8a8; background: rgba(200,184,136,0.1); }
.header .brand { color: #e8d8a8; font-weight: bold; font-size: 14px; }
```

## Agent / Character Visualization (CSS Card Style)
```html
<div class="agent-card">
  <div class="agent-sprite">🔮</div>
  <div class="agent-name">🧙 Quant Analyst</div>
  <div class="agent-title">เทคนิคัล วิซาร์ด</div>
  <span class="agent-status active">🟢 WORKING</span>
  <div class="hp-bar"><div class="hp-fill" style="width:85%"></div></div>
  <div style="display:flex;justify-content:space-between;font-size:7px">
    <span>LV.8</span><span>HP 85/100</span>
  </div>
</div>
```

## Frontend Data Fetching Pattern

```javascript
async function fetchAPI(url) {
  const resp = await fetch(url);
  return resp.json();
}

async function loadDashboard() {
  const [data1, data2] = await Promise.all([
    fetchAPI('/api/overview'),
    fetchAPI('/api/agents')
  ]);
  renderAll(data1, data2);
}

loadDashboard();
setInterval(loadDashboard, 30000); // Auto-refresh every 30s
```

## Responsive Layout

Use CSS Grid for the main layout:
```css
.container { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
.panel-full { grid-column: 1 / -1; }  /* Full-width panels */
@media (max-width: 768px) { .container { grid-template-columns: 1fr; } }
```

## Extracting Design Details from Reference Images

When the user sends a screenshot of a design they want you to replicate but you cannot see images directly:

1. **Navigate to the image URL** using `browser_navigate`
2. **Extract pixel data via browser console canvas**:
```javascript
var img = document.querySelector('img');
var c = document.createElement('canvas'), ctx = c.getContext('2d');
c.width = img.naturalWidth; c.height = img.naturalHeight;
ctx.drawImage(img, 0, 0);
// Sample specific coordinates
var px = ctx.getImageData(x, y, 1, 1).data;
console.log('rgb(' + px[0] + ',' + px[1] + ',' + px[2] + ')');
```
3. **Scan for layout dividers**: sample the full width at a given Y to find where column boundaries are.
4. **Extract key color swatches**: sample floor, walls, desks, characters, terminal panel, buttons.
5. **Measure proportions**: image dimensions encode the layout ratio (e.g., 640×288 → 62% office / 38% panel).
6. **Use the extracted palette** to rebuild the design accurately.

## Using Real Pixel Art Sprite Sheets

When the user references the [pixel-agents-hq/pixel-agents](https://github.com/pixel-agents-hq/pixel-agents) repo, they expect **actual pixel art sprites** — not colored rectangles. Clone the repo and extract its assets.

### Setup — Copy Sprites

```bash
git clone https://github.com/pixel-agents-hq/pixel-agents.git /tmp/pixel-agents
mkdir -p /root/project/static/assets/{characters,floors,furniture/DESK,furniture/PC,furniture/WOODEN_CHAIR,furniture/LARGE_PLANT}
cp /tmp/pixel-agents/webview-ui/public/assets/characters/*.png /root/project/static/assets/characters/
cp /tmp/pixel-agents/webview-ui/public/assets/floors/floor_0.png /root/project/static/assets/floors/
cp /tmp/pixel-agents/webview-ui/public/assets/furniture/DESK/DESK_FRONT.png /root/project/static/assets/furniture/DESK/
cp /tmp/pixel-agents/webview-ui/public/assets/furniture/PC/PC_FRONT_ON_1.png /root/project/static/assets/furniture/PC/
cp /tmp/pixel-agents/webview-ui/public/assets/furniture/WOODEN_CHAIR/WOODEN_CHAIR_FRONT.png /root/project/static/assets/furniture/WOODEN_CHAIR/
cp /tmp/pixel-agents/webview-ui/public/assets/furniture/LARGE_PLANT/LARGE_PLANT.png /root/project/static/assets/furniture/LARGE_PLANT/
```

### Sprite Dimensions

| Asset | Size (px) | Notes |
|-------|-----------|-------|
| Character spritesheet | 112 × 96 | 7 cols × 16px, 6 rows × 16px (= 7 frames × 6 directions) |
| Floor tile | 16 × 16 | Tileable square |
| Desk (front) | 48 × 32 | Wide desk viewed from front |
| PC (front, on) | 16 × 32 | CRT monitor with green screen |
| Wooden chair (front) | 16 × 32 | Chair viewed from front |
| Potted plant | 32 × 48 | Decorative large plant |

### Spritesheet Structure

Character spritesheets encode **7 animation columns** × **6 direction/activity rows** (each frame 16×16):

| Row | Dir/Activity | Frame 0 | Frame 1 | Frame 2 | Frame 3 | Frame 4 | Frame 5 | Frame 6 |
|-----|-------------|---------|---------|---------|---------|---------|---------|---------|
| 0 | Down (front) | walk-0 | walk-1 | walk-2 | walk-3 | idle-0 | idle-1 | idle-2 |
| 1 | Left | walk-0 | walk-1 | walk-2 | walk-3 | idle-0 | idle-1 | idle-2 |
| 2 | Right | walk-0 | walk-1 | walk-2 | walk-3 | idle-0 | idle-1 | idle-2 |
| 3 | Up (back) | walk-0 | walk-1 | walk-2 | walk-3 | idle-0 | idle-1 | idle-2 |
| 4 | Activity A | frame-0 | frame-1 | frame-2 | frame-3 | frame-4 | frame-5 | frame-6 |
| 5 | Activity B | frame-0 | frame-1 | frame-2 | frame-3 | frame-4 | frame-5 | frame-6 |

**⚠️ Note:** The exact row-to-direction mapping varies by spritesheet. Always verify by inspecting pixel content (check if frame 0 of adjacent rows has similar color profiles — same character, different pose). The `ch` sheet variant in pixel-agents has different column/row counts. When in doubt, sample first frame of each row to determine if rows are directions or different characters.

#### Drawing Sprites from the Sheet

When rendering from a pixel-agents spritesheet, the row represents **direction/activity** and the column represents **animation frame**:

```javascript
var ss = 16; // sprite cell size
var scale = 3; // pixel scaling factor

// charIdx: which character spritesheet (0-5)
// dir: direction row (0=down, 1=left, 2=right, 3=up)
// frame: animation column (0-6)
function drawSprite(ctx, img, charIdx, dir, frame, x, y, scale) {
  ctx.drawImage(
    img,           // the char_N.png Image
    frame * ss,    // source X: animation column
    dir * ss,      // source Y: direction row
    ss, ss,        // source size (16×16)
    x, y,          // destination position
    ss * scale,    // destination width
    ss * scale     // destination height
  );
}
```

**Frame mapping (preferred approach — match pixel-agents repo):**
| State | Direction Row | Frame Columns | Description |
|-------|--------------|---------------|-------------|
| Walk | 0-3 (dir) | 0, 1, 2, 3 | 4-frame walk cycle |
| Type | 4 | 0, 1 | 2-frame typing at desk |
| Read | 5 | 0, 1 | 2-frame reading animation |
| Idle | 0-3 (dir) | 6 | Single idle pose |

**⚠️ CRITICAL — Sprite stretching pitfall:** The most common user complaint ("จอมันยาวไป / monitor is too long") comes from rendering the PC sprite (16×32 = portrait, 1:2 aspect ratio) at the wrong scale. Each sprite has a **fixed native aspect ratio**. Never force-fit into a square tile unless the sprite is square. Render as:
- PC: 1 tile wide × 2 tiles tall (16×32 → 48×96 at 3x scale)
- Desk: 3 tiles wide × 2 tiles tall (48×32 → 144×96)
- Chair: 1 tile wide × 2 tiles tall (16×32 → 48×96)
- Plant: 2 tiles wide × 3 tiles tall (32×48 → 96×144)

**Drawing a sprite from the sheet:**
```javascript
ctx.drawImage(spriteSheet, frame * 16, dir * 16, 16, 16, canvasX, canvasY, 32, 32);
```
Use `ctx.imageSmoothingEnabled = false` to keep pixel-perfect rendering.

### Python Server — Serving Static Sprites

Add a `/static/` route to the HTTP handler:

```python
elif p.startswith("/static/"):
    fpath = os.path.join(ROOT, p.lstrip("/"))
    if os.path.exists(fpath) and os.path.isfile(fpath):
        self.send_response(200)
        if p.endswith(".png"): self.send_header("Content-Type", "image/png")
        self.send_header("Cache-Control", "max-age=3600")
        self.end_headers()
        with open(fpath, "rb") as f: self.wfile.write(f.read())
    else: self.send_error(404)
```

### Client-Side Sprite Loading

```javascript
var sprites = {};
var loaded = 0;
var TOTAL = 8;

function loadSprite(name, src, callback) {
  var img = new Image();
  img.onload = function() {
    sprites[name] = img;
    loaded++;
    if (loaded >= TOTAL && callback) callback();
  };
  img.onerror = function() { loaded++; };
  img.src = src;
}

// Load all sprites
loadSprite('floor', '/static/assets/floors/floor_0.png');
loadSprite('desk_front', '/static/assets/furniture/DESK/DESK_FRONT.png');
for (var i = 0; i < 6; i++)
  loadSprite('char_' + i, '/static/assets/characters/char_' + i + '.png');

// Fallback start timer
setTimeout(startGame, 2000);
```

### Activity-to-Direction Mapping

```javascript
function getSpriteFrame(agent) {
  if (agent.activity === 'typing' || agent.activity === 'analyzing') {
    return { dir: 4, frame: Math.floor(Date.now() / 500) % 2 };  // typing row
  } else if (agent.activity === 'reading') {
    return { dir: 5, frame: Math.floor(Date.now() / 600) % 2 };  // reading row
  } else if (agent.activity === 'walking') {
    return { dir: 0, frame: agent.wf % 4 };  // walk frames
  } else {
    return { dir: 0, frame: Math.floor(Date.now() / 800) % 2 };  // idle breathing
  }
}
```

## Pitfalls

- **Font CDN dependency**: `Press Start 2P` from Google Fonts may not load in offline/air-gapped environments. Host it locally or use `VT323` / `Monospace` as fallback.
- **CRT scan lines can cause accessibility issues**: users with visual sensitivities may find the flicker disturbing. Make it optional (toggle button or prefers-reduced-motion media query).
- **http.server is single-threaded**: for production use, wrap in gunicorn/uWSGI or use Flask/FastAPI. For lightweight internal dashboards (< 5 concurrent users), http.server is fine.
- **Large datasets**: sending 10,000 rows of trade data via JSON will bloat the page. Paginate at the API level (limit=50) and load more on scroll.
- **CORS**: the `Access-Control-Allow-Origin: *` header is required if the frontend JS fetches from a different origin. For same-origin serving, it's optional.
- **BBR warm palette needs careful contrast**: light text on `#3d3224` headers is readable, but ensure text on `#fff8ee` panels stays `#3d3224` (not lighter) to meet WCAG AA.
- **Canvas animation stops when tab is hidden**: browsers throttle requestAnimationFrame to 0 when the tab isn't visible. This is fine for dashboards — the simulation catches up when the user returns.
- **Agent activity timing**: agents switch activities every 100-300 frames. If this feels too fast or too slow, adjust the `activityTimer` threshold.
- **Sprite stretching (most common complaint)**: each sprite has a fixed native size. A PC sprite (16x32 = portrait) will look "long" if rendered at 1:1 aspect ratio but expected to be square. Always check the **native pixel dimensions** of each sprite before rendering — render at `spriteWidth × scale` by `spriteHeight × scale`, never force-fit into a square. Key sprites: PC=16x32 (portrait), Desk=48x32 (landscape), Chair=16x32 (portrait), Plant=32x48 (portrait). If the user says "monitor is too long", the sprite is being rendered at the wrong aspect ratio.
- **Character sitting position (most overlooked)**: when a character sits at a desk, they must be positioned at the **TOP** tile row of the desk area, NOT the center. Example: if a desk occupies tiles rows 3-4 (2 tiles high), place the character at row 3, not row 4. At row 3, the character's sprite starts above the desk, their lower body is covered by the desk image → looks like sitting. At row 4, the character's ENTIRE body overlaps with the desk → invisible behind it or floating on top. See the Z-sorted rendering section above for the correct approach.
- **Sitting offset is mandatory**: Even when the character is at the correct row (top of desk area), you MUST shift their sprite down by 12px (at 3x scale) to make them look seated. Without this, too much of the character's body shows above the desk and they still look like they're standing behind it.
- **Spritesheet row mapping varies**: The pixel-agents spritesheets have rows 0-3 = directional walks (down, left, right, up) and rows 4-5 = activity animations (typing, reading). Don't assume all rows are directions. Verify by comparing pixel content of frame 0 across rows.
- **Canvas + side panel layout**: When using flex layout for canvas + side panel, ensure the canvas resizes when the container resizes (window resize, tab switch). Use `ResizeObserver` or a manual resize handler triggered by tab activation.

## References

- `references/dashboard-example.md` — full working example from a trading bot dashboard (Pixel Art style, 6 AI agents, monthly P&L chart, trade table, security panels)
- `references/agent-hq-implementation.md` — multi-page server with canvas-based pixel agent animation: walking characters, office environment, speech bubbles, activity log
- `references/bbr-palette.md` — BBR warm palette exact measurements and layout dimensions extracted from image analysis
- `references/light-pixel-tabs.md` — pattern for light-theme dashboard default + hidden pixel art canvas tab with lazy initialization
- `references/z-sorted-pixel-agents.md` — concrete, verified implementation of Z-sorted pixel agent office with sitting offset, typing animation, and walk+return scheduler