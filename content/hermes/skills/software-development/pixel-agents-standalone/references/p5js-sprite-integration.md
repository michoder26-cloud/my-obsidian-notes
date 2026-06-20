# p5.js Lightweight Sprite Integration for Pixel Agents

Reference for injecting real pixel-agents character sprites into an existing p5.js office scene without building the full React+Vite app.

## Context

Some users have a custom p5.js "Pixel Agent Office" running on a Python HTTP server (port 9120). The scene draws desks, monitors, neon signs, bookshelves, and procedural characters via `rect()`/`ellipse()`. The goal is to replace the procedural characters with real sprite sheets from `pixel-agents-hq/pixel-agents` while keeping the p5.js infrastructure intact.

## Why this approach vs. full React build

| Approach | Complexity | Pros | Cons |
|---|---|---|---|
| **p5.js sprite injection** (this doc) | Low — single HTML file, no build step | Fast, keeps existing scene code, keeps Hermes API fetch logic | No wandering/pathfinding, no furniture placement editor, manual frame animation |
| **React+Vite standalone** (main skill) | High — full build pipeline, transpilation | Full office engine: pathfinding, furniture editor, Z-sorting, speech bubbles | Heavy setup, breaks existing custom scene code |

## Sprite Sheet Anatomy (from pixel-agents)

Each `char_N.png` is:
- **112×96 pixels** = 7 frames × 16px wide, 3 directions × 32px tall
- Directions per row: **DOWN** (y=0), **UP** (y=32), **RIGHT** (y=64)
- Frame layout per direction:
  - `0,1,2` = walk frames (0=stand, 1=step, 2=step)
  - `3,4` = typing frames
  - `5,6` = reading frames

```
Sheet layout (112×96):
┌─────────────────────────────────────┬───────────┐
│ down walk0 │ walk1 │ walk2 │ type0 │ type1 │ read0 │ read1 │  y=0..31
├─────────────────────────────────────┼───────────┤
│ up   walk0 │ walk1 │ walk2 │ type0 │ type1 │ read0 │ read1 │  y=32..63
├─────────────────────────────────────┼───────────┤
│ right walk0│ walk1 │ walk2 │ type0 │ type1 │ read0 │ read1 │  y=64..95
└─────────────────────────────────────┴───────────┘
         x=0..15    .. 31   .. 47   .. 63   .. 79   .. 95   .. 111
```

> LEFT direction does not exist in the sheet — generate it at runtime by flipping RIGHT sprites horizontally (`scale(-1, 1)`).

## p5.js Preload & Setup

```javascript
const SPRITE_W = 16;
const SPRITE_H = 32;
const FRAMES_PER_DIR = 7;
const DIR_DOWN = 0, DIR_UP = 1, DIR_RIGHT = 2, DIR_LEFT = 3;

let charSheets = [];  // p5.Image[]
let spriteReady = false;

function preload(){
  for(let i = 0; i < 4; i++){
    charSheets[i] = loadImage('assets/characters/char_' + i + '.png');
  }
}

function setup(){
  // ... existing setup ...
  noSmooth();
  pixelDensity(1);  // critical for crisp pixel art scaling
  spriteReady = true;
}
```

> `noSmooth()` + `pixelDensity(1)` are **essential**. Without them, p5.js anti-aliases the scaled sprites and destroys the pixel-art look.

## Drawing a Single Sprite Frame

```javascript
function getSpriteFrame(sheetIdx, dir, frameIdx){
  const sheet = charSheets[sheetIdx];
  if(!sheet) return null;
  const row = (dir === DIR_LEFT) ? DIR_RIGHT : dir;  // left reuses right row
  const sx  = (frameIdx % FRAMES_PER_DIR) * SPRITE_W;
  const sy  = row * SPRITE_H;
  return {img: sheet, sx, sy, sw: SPRITE_W, sh: SPRITE_H};
}
```

Usage in draw:

```javascript
const frame = getSpriteFrame(charIdx, DIR_DOWN, 3); // typing frame 0
if(frame){
  // ⚠️ p5.js image() does NOT support source-rect (9-arg form)!
  // Use drawingContext.drawImage() with the p5.Image's .canvas property:
  drawingContext.drawImage(frame.img.canvas,
    frame.sx, frame.sy, frame.sw, frame.sh,       // source rect
    drawX, drawY, SPRITE_W*4, SPRITE_H*4);         // dest rect
}
```

> Scale factor of **4×** is a sweet spot: 16×32 becomes 64×128, big enough to see detail, small enough to fit 4 agents across a 1280px canvas.

## Horizontal Flip for LEFT Direction

```javascript
push();
translate(drawX + SPRITE_W * scale, drawY);
scale(-1, 1);
// Use drawingContext.drawImage, NOT p5 image() — see note above
drawingContext.drawImage(frame.img.canvas,
  frame.sx, frame.sy, frame.sw, frame.sh,
  0, 0, SPRITE_W * scale, SPRITE_H * scale);
pop();
```

Or via CSS on the canvas (if you never need to flip dynamically):

```css
canvas { image-rendering: pixelated; image-rendering: crisp-edges; }
```

## Animation State Machine (for p5.js)

Match the pixel-agents animation timings approximately:

```javascript
function getFrameForState(agent, st){
  agent.frameTimer++;

  if(st === 'offline'){
    // frozen standing pose
    return getSpriteFrame(agent.charIdx, agent.dir, 1);
  }
  if(agent.state === 'type'){
    const period = 6;  // frames per typing frame
    const typeFrame = Math.floor(agent.frameTimer / period) % 2;
    return getSpriteFrame(agent.charIdx, agent.dir, 3 + typeFrame);
  }
  if(agent.state === 'walk'){
    const period = 8;
    const walkFrame = Math.floor(agent.frameTimer / period) % 4;
    const mapped = [0, 1, 2, 1][walkFrame];  // 4-frame cycle with backstep
    return getSpriteFrame(agent.charIdx, agent.dir, mapped);
  }
  // idle
  return getSpriteFrame(agent.charIdx, agent.dir, 1);
}
```

## Integration into existing scene

Replace the existing `drawCharacter()` call with `drawSpriteCharacter()`. Keep everything else (desk, monitor, coffee, nameplate, dust, stars, background) untouched.

```javascript
function drawStation(s){
  // ... update timers ...
  // Character (sprite-based, behind desk)
  drawSpriteCharacter(s, x, y, st);
  // Desk (drawn after character to occlude lower body)
  drawDesk(x, y, th);
  // Monitor (occludes torso)
  drawMonitor(s, x, y - 50, st);
  // ...
}
```

> The sprite is drawn at `finalY = y - SPRITE_H*scale + offset` so the character feet sit naturally relative to the desk surface.

## Where to copy sprite sheets from

Clone the pixel-agents repo once, then copy sheets into your static assets:

```bash
git clone --depth 1 https://github.com/pixel-agents-hq/pixel-agents.git /tmp/pixel-agents
cp /tmp/pixel-agents/webview-ui/public/assets/characters/char_{0,1,2,3,4,5}.png \
   /path/to/your/server/assets/characters/
```

6 sheets ship with the repo by default. They are pre-colored (each sheet is a different palette — no runtime palette swap needed).

## Known gotchas

1. **`image()` does NOT support source-rect (9-arg form) — use `drawingContext.drawImage()`** — p5.js `image()` only accepts `image(img, x, y)` or `image(img, x, y, w, h)`. There is NO 9-argument source-rect overload. If you pass 9 args, it silently no-ops and nothing renders. **The fix:** use the raw canvas context's `drawImage()`, which DOES support the 9-arg form: `drawingContext.drawImage(p5Image.canvas, sx, sy, sw, sh, dx, dy, dw, dh)`. Access the underlying canvas via the `.canvas` property of the p5.Image wrapper. The p5 transform stack (`push()`/`translate()`/`scale()`) still applies because `drawImage` honors the current `drawingContext` transform matrix.
2. **`drawingContext.filter`** — p5.js wraps the native canvas context. Setting `drawingContext.filter = 'brightness(0.35) grayscale(0.8)'` works, but you MUST reset it with `drawingContext.filter = 'none'` before the next draw call or all subsequent sprites will be darkened.
3. **p5.Image CAN be passed to raw `ctx.drawImage()` via `.canvas`** — `loadImage()` returns a p5.Image wrapper. To use it with `drawingContext.drawImage()`, pass `.canvas` (the underlying `HTMLCanvasElement`). Confirmed working in p5.js 1.11.3.
4. **Missing `noSmooth()`** — Without this, scaled sprites get bilinear interpolation and lose their sharp edges. Always set it immediately after `createCanvas()`.
5. **Pixel density on retina displays** — `pixelDensity(1)` forces 1:1 physical pixels, preventing p5 from doubling resolution on HiDPI screens which makes 16px sprites even tinier.
6. **⚠️ Variable shadowing of p5.js built-in functions — THE #1 RECURRING BUG** — This has now hit across MULTIPLE sessions. Never name a variable, local, or function parameter the same as ANY p5.js global. The complete known trap list with confirmed renames:

   | Shadowed p5 function | Bad name | Safe rename (confirmed working) |
   |---|---|---|
   | `text()` | `text` (param or var) | `signText`, `labelText`, `msgText` |
   | `scale()` | `scale` (const/local) | `spriteScale`, `drawScale`, `zoom` |
   | `line()` | `line` (var holding object/string) | `codeLine`, `lineData`, `ln` |
   | `fill()` | `fill` | `fillColor`, `bgFill` |
   | `image()` | `image` | `img`, `sprite` |

   **When it hits:** the error is `TypeError: X is not a function` or `TypeError: line.includes is not a function` (for object-typed vars). It kills the `draw()` loop silently after frame 1 — p5 catches the exception and stops. **No console error appears.** This is the most insidious p5.js bug because the page looks fine (background renders on frame 1) but nothing animates.

   **Quick diagnosis via browser console:**
   ```javascript
   // 1. Check if draw loop is frozen
   JSON.stringify({drawCallCount, spriteDrawCount})
   // If drawCallCount === 1, the loop died after frame 1

   // 2. Call each sub-function in try/catch to find the throw
   const errors = [];
   try { drawBackground(); } catch(e) { errors.push('drawBackground: ' + e.message); }
   try { drawOfficeDecor(); } catch(e) { errors.push('drawOfficeDecor: ' + e.message); }
   try { for(const s of stations) drawStation(s); } catch(e) { errors.push('drawStation: ' + e.message); }
   JSON.stringify({errors, drawCallCount});
   // Example output: {"errors": ["drawOfficeDecor: text is not a function", "drawStation: line.includes is not a function"]}
   ```

   **Object property access gotcha:** if `line` is an object like `{text: "...", y: 42}`, then `line.includes(...)` throws "line.includes is not a function" — not because `includes` is shadowed, but because the object doesn't have that method. The fix is `line.text.includes(...)`.

7. **Uncaught exceptions in `draw()` kill the animation loop silently** — p5.js catches exceptions in `draw()` and stops calling it. There is no console error. Add `drawCallCount` counter, or wrap sub-functions in try/catch during debugging to isolate the throw. See gotcha #6 above for the diagnostic recipe — it has been used successfully across multiple sessions.
