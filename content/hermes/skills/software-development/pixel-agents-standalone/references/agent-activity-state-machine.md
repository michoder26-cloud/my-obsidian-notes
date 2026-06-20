# Agent Activity State Machine — CPU-Driven Behavior

Reference for making pixel office agents walk between a workstation and a lounge based on real-time Hermes API CPU data.

## Concept

Each agent has two locations:
- **Workstation** (desk + monitor) — where they sit and type when busy
- **Lounge** (sofa/rest area) — where they idle/wander when not busy

The agent's location is driven by its real CPU usage from the Hermes `/api/status` endpoint. High CPU = working at desk; low CPU = resting at lounge. This creates a living office where you can see at a glance which agents are active.

## State Machine

```
resting ──(CPU > 2% for 1s)──> to_desk ──(arrived)──> working
    ^                                                          │
    └───(arrived)──── to_rest ◄──(CPU < 0.8% for 10s)──────────┘
```

Four states per agent:
| State | Description | Character animation |
|---|---|---|
| `resting` | At lounge, gentle wander | `idle` or slow `walk` |
| `to_desk` | Walking from lounge to desk | `walk` |
| `working` | At desk, typing | `type` |
| `to_rest` | Walking from desk to lounge | `walk` |

## CPU Thresholds

```javascript
const CPU_ACTIVE = 2.0;    // CPU% above this = agent is busy
const CPU_IDLE = 0.8;      // CPU% below this = agent is idle
const ACTIVE_HOLD = 60;    // frames to confirm active (~1s at 60fps)
const IDLE_HOLD = 600;     // frames to confirm idle (~10s at 60fps)
```

> The hysteresis (ACTIVE_HOLD / IDLE_HOLD) prevents jitter: an agent won't jump up for a 1-frame CPU spike, and won't immediately leave the desk during a brief lull. Tune `IDLE_HOLD` lower if agents should leave the desk faster.

## Station Data Model

Each station object needs both location sets plus current animated position:

```javascript
stations.push({
  // Desk position (workstation)
  deskX: 140 + i*300,  deskY: 300,
  // Rest position (lounge)
  restX: restSpots[i].rx,  restY: restSpots[i].ry,
  // Current render position (interpolated)
  curX: restSpots[i].rx,  curY: restSpots[i].ry,
  // Target position (where walking to)
  targetX: restSpots[i].rx,  targetY: restSpots[i].ry,
  // FSM state
  activity: 'resting',     // 'resting' | 'to_desk' | 'working' | 'to_rest'
  state: 'idle',           // 'idle' | 'walk' | 'type' (sprite animation)
  dir: DIR_DOWN,           // current facing direction
  // CPU tracking
  activeTimer: 0,  idleTimer: 0,
  // Wander at rest
  wanderTimer: 0,  wanderX: restSpots[i].rx,
});
```

## Update Function (called every frame)

```javascript
function updateAgentActivity(s){
  const data = agentData(s.theme.key);
  const cpu = data ? data.cpu : 0;
  const st = agentStatus(s.theme.key);

  if(st === 'offline'){ s.activity = 'resting'; s.state = 'idle'; return; }

  // Accumulate timers
  if(cpu > CPU_ACTIVE){ s.activeTimer++; s.idleTimer = 0; }
  else if(cpu < CPU_IDLE){ s.idleTimer++; s.activeTimer = 0; }

  // State transitions
  switch(s.activity){
    case 'resting':
      if(s.activeTimer > ACTIVE_HOLD){
        s.activity = 'to_desk'; s.targetX = s.deskX; s.targetY = s.deskY;
      }
      break;
    case 'to_desk':
      if(arrived(s)){ s.activity = 'working'; s.state = 'type'; }
      break;
    case 'working':
      if(s.idleTimer > IDLE_HOLD){
        s.activity = 'to_rest'; s.targetX = s.restX; s.targetY = s.restY;
      }
      break;
    case 'to_rest':
      if(arrived(s)){ s.activity = 'resting'; s.state = 'idle'; }
      break;
  }

  // Movement
  if(s.activity === 'to_desk' || s.activity === 'to_rest'){
    moveToward(s, s.targetX, s.targetY);
    s.state = 'walk';
  } else if(s.activity === 'resting'){
    // Gentle wander near rest spot
    s.wanderTimer--;
    if(s.wanderTimer <= 0){
      s.wanderX = s.restX + (Math.random()-0.5)*60;
      s.wanderTimer = Math.random()*400+200;
    }
    moveToward(s, s.wanderX, s.restY, 0.3);  // slower wander
    s.state = (Math.abs(s.curX - s.wanderX) > 2) ? 'walk' : 'idle';
  } else if(s.activity === 'working'){
    s.state = 'type';
  }
}
```

## Movement & Facing Direction

```javascript
function moveToward(s, tx, ty, speedMult){
  speedMult = speedMult || 1.0;
  const dx = tx - s.curX, dy = ty - s.curY;
  const dist = Math.sqrt(dx*dx + dy*dy);
  const speed = 1.8 * speedMult;
  if(dist < speed){ s.curX = tx; s.curY = ty; return; }
  s.curX += (dx/dist) * speed;
  s.curY += (dy/dist) * speed;
  // Face movement direction (horizontal priority for natural look)
  if(Math.abs(dx) > Math.abs(dy)){
    s.dir = dx > 0 ? DIR_RIGHT : DIR_LEFT;
  } else {
    s.dir = dy > 0 ? DIR_DOWN : DIR_UP;
  }
}
```

## Rendering Notes

- **Desk + monitor** are always drawn at `deskX/deskY` regardless of agent position
- **Monitor** goes dark (OFFLINE state) when agent is NOT at desk: `drawMonitor(s, s.deskX, s.deskY-50, s.activity === 'working' ? st : 'offline')`
- **Character** is drawn at `curX/curY` (animated position), NOT at desk position
- **Coffee mug** only appears when agent is at desk
- **Tooltip/hit detection** must use `curX/curY`, not a fixed `x/y`

## Lounge Layout

Place rest spots in a row at the bottom of the canvas (y ≈ 600-630 on a 720px canvas). Draw a procedural sofa with cushions colored per-agent theme, plus plants and a coffee table for atmosphere.

## Server-Side Work Detection (CRITICAL — CPU thresholds alone are unreliable)

The CPU-threshold approach above is fragile because gateway idle CPU (~0.6-0.8%) sits very close to the `CPU_IDLE` threshold. A far more reliable approach is to detect **active work processes** server-side.

### The Problem

`hermes --profile X gateway run` is the long-lived gateway process. When a user sends a task (via Telegram, CLI, or delegation), a SEPARATE `hermes --profile X -z "task"` process spawns. The original `check_gateway()` only matched the gateway process, so it couldn't detect active work — CPU stayed at 0.8% even while an agent was processing a 45% CPU LLM call.

### The Fix: server.py `check_gateway()` modification

```python
def check_gateway(self, profile):
    result = subprocess.run(["ps", "aux", "--no-headers"], ...)
    gw_pid = 0; gw_cpu = 0.0; gw_mem_kb = 0
    work_cpu = 0.0; work_count = 0

    for line in result.stdout.split("\n"):
        if "grep" in line or "python" not in line:
            continue
        # Gateway process
        if f"--profile {profile} gateway" in line:
            parts = line.split()
            gw_pid = int(parts[1]); gw_cpu = float(parts[2]); gw_mem_kb = int(parts[5])
        # Active work process (hermes --profile X but NOT gateway)
        elif f"--profile {profile}" in line and "gateway" not in line:
            parts = line.split()
            work_cpu += float(parts[2])
            work_count += 1

    total_cpu = gw_cpu + work_cpu
    # status = "working" if work_count > 0, else "online"/"warning"
    return {
        "name": profile, "status": agent_status,
        "pid": gw_pid, "cpu": round(total_cpu, 1),
        "mem": f"{mem_mb}MB", "uptime": uptime_str,
        "working": work_count > 0   # <-- the key new field
    }
```

### Client-side: use `working` flag instead of CPU thresholds

```javascript
function updateAgentActivity(s){
  const data = agentData(s.theme.key);
  const isWorking = data && data.working === true;
  const cpu = data ? data.cpu : 0;

  // Use working flag + CPU as fallback
  const ACTIVE_HOLD = 15;    // ~0.25s — very fast response
  const IDLE_HOLD = 120;     // ~2s — leave desk shortly after work ends

  if(isWorking || cpu > 5.0){
    s.activeTimer++; s.idleTimer = 0;
  } else {
    s.idleTimer++; s.activeTimer = 0;
  }
  // ... rest of state machine unchanged
}
```

### Testing: trigger agents to walk to desks

To test the state machine, spawn real hermes CLI tasks (NOT Telegram messages — bots in the same group can't see each other's messages):

```bash
# Run in background — these will spike CPU and set working=True
hermes --profile coder -z "Write a Python script that..." --yolo &
hermes --profile news -z "Summarize today's economic news..." --yolo &
```

Verify via API:
```bash
curl -s http://localhost:9120/api/status | python3 -c "
import sys,json
for a in json.load(sys.stdin)['agents']:
    print(f\"{a['name']:10s} cpu={a['cpu']:5.1f}%  working={a.get('working')}\")"
# Expected: working=True, cpu=15-45% for agents with active tasks
```

## Pitfalls

- **Don't hardcode `s.dir = DIR_DOWN`** in the character draw function — the state machine sets `s.dir` based on movement. If the draw function overrides it, agents will always face down even while walking sideways.
- **Monitor screen state**: when agent leaves desk, set monitor to 'offline' so the screen goes dark. When they return, restore to actual agent status.
- **IDLE_HOLD too short**: agents will ping-pong between desk and lounge if CPU fluctuates around the threshold. 2 seconds (120 frames) is a good default with the `working` flag approach.
- **Telegram group testing doesn't work**: bots in the same Telegram group cannot see each other's messages (privacy mode). To trigger agent activity for testing, use `hermes --profile X -z "task" --yolo` via terminal instead.
- **Gateway vs work process**: `check_gateway()` must scan for BOTH `--profile X gateway` (the long-lived daemon) AND `--profile X` without "gateway" (active CLI work sessions). Only the latter indicates the agent is actively processing.
