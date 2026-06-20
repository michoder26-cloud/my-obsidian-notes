# Session Pitfalls: Pixel Art Sprite Integration into Existing Office

## Session: 2026-06-19 — Integrating pixel-agents-hq sprites into existing p5.js Pixel Agent Office

### What happened
User wanted pixel art sprites from `pixel-agents-hq/pixel-agents` integrated into their EXISTING Pixel Agent Office (port 9120, p5.js canvas, Hermes API-connected). Previous attempts produced:
1. First: pixel-agents React app deployed at `/agent-hq/` (wrong — standalone app, NOT integrated)
2. Second: vanilla canvas with sprite loading but lost original design
3. Third: coder agent failed to produce working sprite integration

### Lessons Learned

#### 1. ALWAYS BACKUP before modifying existing index.html
Before asking a coder agent to modify an existing Pixel Agent Office's `index.html`, create a backup:
```bash
cp index.html index.html.bak.$(date +%s)
```
If the agent fails or produces wrong output, the backup is the only recovery path. Original procedural p5.js design was lost permanently because no backup existed.

#### 2. Specify sprite integration EXACTLY in delegation context
When asking a coder agent to replace p5.js procedural `drawCharacter()` with real sprites, the delegation goal must include EXPLICIT code requirements:
```javascript
const sprites = {};
function preload() {
  for(let i=0; i<4; i++) {
    sprites[i] = loadImage('./assets/characters/char_'+i+'.png');
  }
}
// Then in drawCharacter():
image(sprites[idx], x, y, w, h);
```

Simply saying "use sprites" results in procedural fallback. VERIFY afterwards:
```bash
grep -c 'loadImage.*char_' /root/pixel-agent-office/index.html  # must be >0
```

#### 3. Prefer ENHANCING existing infrastructure over building new
User had working Pixel Agent Office at port 9120 with:
- Hermes API integration (`/api/status`)
- p5.js procedural animation (breathing, blinking, dust, day/night cycle)
- CRT/vignette overlays
- Real-time footer status

Should have enhanced this office by replacing ONLY `drawCharacter()` with sprite images. Instead, multiple attempts rebuilt from scratch, losing features each time.

#### 4. User preference revealed: "หน้าเว็บเก่าดูดีกว่า"
User preferred the ORIGINAL procedural p5.js design. Sprites alone (without the original animation/atmosphere) were worse than the original. This means the integration should have KEPT all original visual effects and only swapped character rendering.

#### 5. Coder agent reliability for pixel art integration
When delegating to a coder agent for pixel art sprite integration:
- Provide the EXACT sprite paths: `./assets/characters/char_{0,1,2,3}.png`
- Provide the EXACT agent positions: `x=120, 420, 720, 1020, y=300`
- Require `preload()` + `loadImage()` + `image()` pattern explicitly
- Include verification step: grep for `loadImage` and `image(` in output
- Ask agent to test by running the server and checking visual output

### Recovery Strategy (if backup is lost)
If the original p5.js procedural design is lost and no backup exists:
1. Check if a running server still serves the old version (cached in browser or proxy)
2. Check `/tmp/` for temp copies written during setup
3. Check `~/.local/share/hermes/` or session cache
4. Reconstruct from the procedural code that was read into context (this session captured lines 1-320 of original index.html)
5. Accept that some original features may be permanently lost — inform user honestly

### Verification Commands
```bash
# Verify sprites are actually loaded
curl -s http://localhost:9120/ | grep -c 'loadImage.*char_'  # should be >0
curl -s http://localhost:9120/ | grep -c 'image(sprites'    # should be >0

# Verify original design elements preserved
curl -s http://localhost:9120/ | grep -c 'drawDust'         # should be >0
curl -s http://localhost:9120/ | grep -c 'cycle'            # should be >0
curl -s http://localhost:9120/ | grep -c 'crt'             # should be >0
curl -s http://localhost:9120/ | grep -c '/api/status'      # should be >0
```
