# Z-Sorted Pixel Agent Office — Working Implementation

Concrete code pattern for rendering pixel-agents-hq characters sitting at desks in a canvas-based Agent HQ. Verified working at 3x scale with TILE=48 on an 20×11 grid.

## Constants & Grid

```javascript
const FW = 16, FH = 32;           // sprite cell size (px)
const SC = 3;                      // scale factor
const TILE = FW * SC;              // = 48px per grid tile
const COLS = 20, ROWS = 11;       // office grid
const OX = 20, OY = 10;           // canvas offset
const SITTING_OFFSET = 12;        // px to shift sprite down when seated
```

## Z-Sorted Draw Loop (Full Rendering)

```javascript
// 1. Floor — always drawn first
for (let y = 0; y < ROWS; y++) for (let x = 0; x < COLS; x++) {
  const px = x * TILE + OX, py = y * TILE + OY;
  if (isWall) { /* draw wall */ }
  else if (isDesk) { /* draw floor tile under desk */ }
  else { /* checkerboard floor */ }
}

// 2. Collect all drawables with Z values
const drawList = [];

// PCs (highest on screen = lowest Z)
desks.forEach(d => {
  const z = (d.y - 2) * TILE;
  drawList.push({z, draw: () => ctx.drawImage(pc, ...)});
});

// Plants
drawList.push({z: 1 * TILE, draw: () => ctx.drawImage(plant, ...)});

// Characters (Z = feet position)
agents.forEach(a => {
  const z = a.py * TILE + TILE;  // feet row
  const px = a.px * TILE + OX;
  const py = a.py * TILE + OY + TILE - FH * SC;
  const sittingOffset = a.sitting ? SITTING_OFFSET : 0;
  const sy = py + sittingOffset;
  const fCol = a.walk ? a.frame % 7 : (a.sitting ? a.frame % 4 : 0);
  const dirRow = a.dir === 1 ? 1 : (a.dir === 2 ? 2 : 0);

  drawList.push({
    z,
    draw: () => {
      ctx.fillStyle = 'rgba(0,0,0,0.25)';
      ctx.fillRect(px + 2, sy + FH * SC - 3, FW * SC - 4, 3);
      if (img && img.complete) {
        ctx.drawImage(img, fCol * FW, dirRow * FH, FW, FH, px, sy, FW * SC, FH * SC);
      }
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 9px monospace';
      ctx.textAlign = 'center';
      ctx.fillText(a.name, px + FW * SC / 2, sy - 4);
      // Typing sparkles
      if (a.sitting) {
        const spark = Math.sin(a.idleOff * 8) * 0.5 + 0.5;
        ctx.fillStyle = `rgba(0,255,136,${spark * 0.6})`;
        ctx.fillRect(px + FW * SC / 2 - 2, sy + FH * SC - 20, 4, 2);
        ctx.fillRect(px + FW * SC / 2 - 1, sy + FH * SC - 16, 2, 2);
      }
    }
  });
});

// Desks (Z = +0.1 to sort AFTER characters at same row)
desks.forEach(d => {
  const z = (d.y + 2) * TILE + 0.1;
  drawList.push({z, draw: () => ctx.drawImage(desk, ...)});
});

// Chairs (lowest on screen = highest Z)
desks.forEach(d => {
  const z = (d.y + 3) * TILE;
  drawList.push({z, draw: () => ctx.drawImage(chair, ...)});
});

// 3. Sort + draw
drawList.sort((a, b) => a.z - b.z);
drawList.forEach(item => item.draw());
```

## Agent State Machine

```javascript
const agents = [
  {sheet:0, name:'Quant',  col:3, row:3},
  {sheet:1, name:'News',   col:8, row:3},
  {sheet:2, name:'Bull',   col:13,row:3},
  {sheet:3, name:'Bear',   col:3, row:7},
  {sheet:4, name:'CEO',    col:8, row:7},
  {sheet:5, name:'Learning',col:13,row:7},
].map(a => ({
  ...a, px:a.col, py:a.row, dir:0, frame:0, ft:0,
  walk:false, path:[], sitting:true, idleOff:Math.random()*100
}));
```

## Update Loop

```javascript
agents.forEach(a => {
  if (a.walk && a.path.length) {
    const [nx, ny] = a.path[0];
    const sp = 0.04;
    const dx = nx - a.px, dy = ny - a.py;
    if (Math.abs(dx) < sp && Math.abs(dy) < sp) {
      a.px = nx; a.py = ny; a.path.shift();
      if (!a.path.length) { a.walk = false; a.frame = 0; a.sitting = true; }
    } else {
      a.sitting = false;
      a.px += Math.sign(dx) * sp; a.py += Math.sign(dy) * sp;
      a.dir = Math.abs(dx) > Math.abs(dy) ? (dx > 0 ? 2 : 3) : (dy > 0 ? 0 : 1);
    }
    a.ft += 1/60;
    if (a.ft > 0.12) { a.frame = (a.frame + 1) % 7; a.ft = 0; }
  } else {
    a.idleOff += 1/60;
    if (a.sitting) {
      a.ft += 1/60;
      if (a.ft > 0.3) { a.frame = (a.frame + 1) % 4; a.ft = 0; }
    }
  }
});
```

## Walk + Return Scheduler

```javascript
const targets = [[3,2],[8,2],[13,2],[3,6],[8,6],[13,6],[17,5],[5,9],[10,9],[1,5],[1,9]];
agents.forEach((a, i) => {
  setTimeout(() => {
    // Random wander
    setInterval(() => {
      if (!a.walk && Math.random() < 0.3) {
        const t = targets[Math.floor(Math.random() * targets.length)];
        const path = findPath(Math.round(a.px), Math.round(a.py), t[0], t[1]);
        if (path.length > 1) { a.path = path; a.walk = true; a.sitting = false; }
      }
    }, 3000 + Math.random() * 3000);
    // Return to desk
    setInterval(() => {
      if (!a.walk && !a.sitting) {
        const path = findPath(Math.round(a.px), Math.round(a.py), a.col, a.row);
        if (path.length > 1) { a.path = path; a.walk = true; }
      }
    }, 5000 + Math.random() * 4000);
  }, i * 800);
});
```

## Key Dimensions

| Item | Grid Position | Visual Effect |
|------|--------------|---------------|
| Desk | `{y:3, h:2}` | Covers rows 3-4 |
| Character | `row:3` | Top of desk (head shows above, body hidden) |
| PC | `row:1` | 2 tiles above desk (behind character) |
| Chair | `row:5` | 2 tiles below desk (in front of character) |