# Spawned Agent → Web Dashboard Pattern

How to make `delegate_task`/spawned agents appear on a web dashboard in real time.

## Architecture

```
Hermes (agent loop)
  │
  ├─ delegate_task(goal="...")    ← spawns subagent
  │   └─ registers via POST /api/register-agent  ← writes to spawned_agents.json
  │
  └─ Web Dashboard (separate Flask/HTTP server)
      ├─ GET /api/spawned-agents  ← reads from spawned_agents.json
      └─ Shows cards: name, task, status (WORKING/DONE)
```

## Components

### 1. Shared State File

```
/path/to/project/spawned_agents.json
```

Simple JSON array, no DB needed. Agents older than 1 hour auto-clean.

```json
[
  {
    "id": 1,
    "name": "Scanner Agent",
    "task": "Analyzing XAU/USD patterns",
    "type": "hermes",
    "status": "working",
    "created_at": 1700000000.0,
    "started_at": "14:30:00"
  }
]
```

### 2. API Endpoints (in dashboard server)

**GET `/api/spawned-agents`** — returns `{"agents": [...], "count": N}`

Implementation pattern (Python `http.server`):
```python
def get_spawned_agents(self) -> dict:
    path = "/path/to/spawned_agents.json"
    try:
        with open(path) as f:
            agents = json.load(f)
    except:
        agents = []
    # Auto-clean agents older than 1 hour
    now = time.time()
    alive = [a for a in agents if now - a.get("created_at", 0) < 3600]
    return {"agents": alive, "count": len(alive)}
```

**POST `/api/register-agent`** — registers a new agent
```python
entry = {
    "id": len(agents) + 1,
    "name": name,
    "task": task,
    "type": agent_type,
    "status": "working",
    "created_at": time.time(),
    "started_at": datetime.now().strftime("%H:%M:%S")
}
```

**POST `/api/complete-agent`** — marks agent as done
```python
{"id": agent_id}  # in request body
# → sets a["status"] = "completed"
```

### 3. Frontend (HTML/JS)

```javascript
function renderSpawned() {
  fetch('/api/spawned-agents')
    .then(r => r.json())
    .then(data => {
      const agents = data.agents || [];
      if (agents.length === 0) {
        grid.innerHTML = '<p>No active sub-agents</p>';
        return;
      }
      grid.innerHTML = agents.map(a => `
        <div class="agent-card">
          <div class="name">${a.name}</div>
          <div class="task">${a.task}</div>
          <span class="badge ${a.status === 'working' ? 'working' : 'done'}">
            ${a.status === 'working' ? 'WORKING' : 'DONE'}
          </span>
        </div>
      `).join('');
    });
}
```

### 4. Registering from delegate_task

When spawning a subagent via `delegate_task`, register it first:

```python
# Inside execute_code or terminal:
import requests
requests.post("http://localhost:8080/api/register-agent", json={
    "name": "Scanner",
    "task": "Analyzing patterns",
    "type": "hermes"
})
```

Or with curl:
```bash
curl -X POST http://localhost:8080/api/register-agent \
  -H "Content-Type: application/json" \
  -d '{"name":"Scanner","task":"Analysis","type":"hermes"}'
```

Then spawn the agent:
```python
delegate_task(goal="Analyze XAU/USD...")
```

When done, complete:
```python
requests.post("http://localhost:8080/api/complete-agent", json={"id": 1})
```

## Auto-Refresh

Dashboard polls every 15 seconds:
```javascript
setInterval(loadDashboard, 15000);
```

Stale agents (older than 1 hour) are silently removed from the JSON file on every GET request.