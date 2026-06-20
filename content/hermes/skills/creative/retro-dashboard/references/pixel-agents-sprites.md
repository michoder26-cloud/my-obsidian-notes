# pixel-agents-hq/pixel-agents — Sprite Asset Reference

Source repo: https://github.com/pixel-agents-hq/pixel-agents

This is a VS Code extension that renders AI agents as pixel-art characters in an office environment. Its `webview-ui/public/assets/` directory contains production-quality 16×16 pixel art sprites usable in any web-based pixel agent dashboard.

## Asset Directory Structure

```
webview-ui/public/assets/
├── characters/
│   ├── char_0.png  ... char_5.png   (6 character spritesheets, 112×96 each)
├── floors/
│   ├── floor_0.png  ... floor_8.png (tileable 16×16 floor textures)
├── furniture/
│   ├── DESK/
│   │   ├── DESK_FRONT.png (48×32)
│   │   └── DESK_SIDE.png  (16×64)
│   ├── PC/
│   │   ├── PC_FRONT_ON_1.png (16×32)
│   │   ├── PC_FRONT_ON_2.png (16×32)
│   │   ├── PC_FRONT_ON_3.png (16×32)
│   │   ├── PC_FRONT_OFF.png  (16×32)
│   │   ├── PC_BACK.png       (16×32)
│   │   └── PC_SIDE.png       (16×32)
│   ├── WOODEN_CHAIR/
│   │   ├── WOODEN_CHAIR_FRONT.png (16×32)
│   │   ├── WOODEN_CHAIR_SIDE.png  (16×32)
│   │   └── WOODEN_CHAIR_BACK.png  (16×32)
│   ├── LARGE_PLANT/LARGE_PLANT.png (32×48)
│   ├── PLANT/PLANT.png             (16×32)
│   ├── SOFA/ (3 views: FRONT, SIDE, BACK)
│   ├── BOOKSHELF/BOOKSHELF.png
│   ├── WHITEBOARD/WHITEBOARD.png
│   ├── CLOCK/CLOCK.png
│   ├── BIN/BIN.png
│   ├── COFFEE/COFFEE.png
│   ├── CACTUS/CACTUS.png
│   ├── POT/POT.png
│   └── various other furniture...
├── walls/
│   └── wall_0.png (wall tiles)
```

## Character Spritesheet Format

Each character PNG is **112×96** RGBA.

- **112 px wide** = 7 columns × 16 px per frame
- **96 px tall** = 6 rows × 16 px per direction

### Row Mapping (Direction / Activity)

| Row | Direction | Purpose | Frame Usage |
|-----|-----------|---------|-------------|
| 0 | Down | Walking/idle front view | frames 0-3 walk, 4-6 idle |
| 1 | Left | Walking/idle left view | frames 0-3 walk, 4-6 idle |
| 2 | Right | Walking/idle right view | frames 0-3 walk, 4-6 idle |
| 3 | Up | Walking/idle back view | frames 0-3 walk, 4-6 idle |
| 4 | Activity A | Activity animation | frames 0-6 (e.g. typing) |
| 5 | Activity B | Activity animation | frames 0-6 (e.g. reading) |

**⚠️ Row-to-direction mapping varies per spritesheet.** Always verify by comparing pixel content of frame 0 across rows. The `ch` variant sheet has a different structure. When rows have different color profiles, they may be separate character models rather than directions.

### Frame Slicing (JavaScript)

```javascript
// Draw frame 2, direction 0 (walking front, 3rd frame)
var sx = 2 * 16;  // 32
var sy = 0 * 16;  // 0
var sw = 16, sh = 16;
var dx = 100, dy = 100; // canvas position
var scale = 2; // 2x for crisp pixel art on retina displays

ctx.imageSmoothingEnabled = false;
ctx.drawImage(charSprite, sx, sy, sw, sh, dx, dy, sw * scale, sh * scale);
```

### Frame Columns vs Character State

| Frame Column | State | Usage |
|-------------|-------|-------|
| 0-3 | Walk | 4-frame walk cycle |
| 4-5 | Type/Read | 2-frame activity animation |
| 6 | Idle | Static idle pose |

### Sizing Pitfall (Critical)

Each sprite has a **fixed native aspect ratio**. Never force-fit into a square tile. Render at native proportions:

| Sprite | Native (px) | Tile grid | At 3x scale |
|--------|------------|-----------|-------------|
| PC | 16×32 | 1 wide × 2 tall | 48×96 |
| Desk front | 48×32 | 3 wide × 2 tall | 144×96 |
| Chair | 16×32 | 1 wide × 2 tall | 48×96 |
| Plant | 32×48 | 2 wide × 3 tall | 96×144 |
| Character | 16×16 | 1×1 | 48×48 |

## Character Index Map

| File | Index | Default Role (from repo) |
|------|-------|--------------------------|
| char_0.png | 0 | Character 1 (base palette) |
| char_1.png | 1 | Character 2 |
| char_2.png | 2 | Character 3 |
| char_3.png | 3 | Character 4 |
| char_4.png | 4 | Character 5 |
| char_5.png | 5 | Character 6 |

The repo supports up to 6 character palettes (each with colorized variants via hue-shift). Characters are derived from JIK-A-4's "Metro City" free top-down character pack (https://jik-a-4.itch.io/metrocity-free-topdown-character-pack).

## Colorizing Characters

The pixel-agents codebase has a `colorize.ts` module that applies hue shifts to character sprites. For standalone use, pre-color the characters using the built-in color variants or create your own colored versions.