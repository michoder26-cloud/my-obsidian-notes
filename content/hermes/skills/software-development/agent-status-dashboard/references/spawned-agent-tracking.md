# Spawned Agent Tracking Implementation

## How it works

A JSON file (`spawned_agents.json`) serves as the shared state between Hermes (who spawns agents via `delegate_task`) and the dashboard web server.

## File format: `spawned_agents.json`

```json
[
    {
        "id": 1,
        "name": "Scanner Agent",
        "task": "Analyzing XAU/USD patterns",
        "type": "hermes",
        "status": "working",
        "created_at": 1781466050.273689,
        "started_at": "21:40:50"
    }
]
```

## API endpoints to add to Python server

```python
# In DashboardAPI class:
def get_spawned_agents(self) -> dict:
    path = os.path.join(os.path.dirname(__file__), "spawned_agents.json")
    try:
        if os.path.exists(path):
            with open(path) as f:
                agents = json.load(f)
        else:
            agents = []
    except:
        agents = []
    # Clean up agents older than 1 hour
    now = datetime.now().timestamp()
    alive = [a for a in agents if now - a.get("created_at", 0) < 3600]
    return {"agents": alive, "count": len(alive)}

def register_agent(self, name, task, agent_type="hermes"):
    # Read, append, write
    # Returns {"ok": True, "id": N}

def complete_agent(self, agent_id):
    # Read, update status to "completed", write
    # Returns {"ok": True}
```

```python
# In DashboardHandler.do_GET():
elif path == "/api/spawned-agents":
    return self._serve_json(self.api.get_spawned_agents())

# In DashboardHandler.do_POST():
if path == "/api/register-agent":
    body = json.loads(...)
    result = self.api.register_agent(body.get("name"), body.get("task"))
    return self._serve_json(result)
elif path == "/api/complete-agent":
    result = self.api.complete_agent(body.get("id", 0))
    return self._serve_json(result)
```

## Dashboard HTML section

```html
<!-- Spawned Agents -->
<div class="section-title">🧬 Active Sub-Agents</div>
<div class="agents-grid" id="spawnedGrid">
    <div class="loading">Checking...</div>
</div>
```

## JavaScript

```javascript
function renderSpawned() {
    fetch('/api/spawned-agents')
        .then(r => r.json())
        .then(data => {
            const agents = data.agents || [];
            if (agents.length === 0) {
                // Show empty state
                return;
            }
            const emojis = ['🔍','📊','🛠️','🧪','📝','🔬','⚙️','💻'];
            agents.map((a, i) => {
                const isWork = a.status === 'working';
                `<div class="agent-card">
                    <div class="agent-avatar">
                        <span class="emoji">${emojis[i % emojis.length]}</span>
                        <span class="status-ring ${isWork ? 'working' : 'idle'}"></span>
                    </div>
                    <div class="agent-info">
                        <div class="name">${a.name}</div>
                        <div class="duty">${a.task}</div>
                        <span class="status-badge ${isWork ? 'working' : 'idle'}">
                            ${isWork ? 'WORKING' : 'DONE'}
                        </span>
                    </div>
                </div>`;
            });
        });
}
```