# Light Theme + Pixel Art Canvas Tabs (June 2026)

## Pattern

Build a dashboard with **light theme as default page** and **pixel art canvas in a hidden tab**, with **lazy canvas initialization** on first tab switch.

### HTML Structure

```html
<!-- Dashboard page (visible by default) -->
<div id="pageDashboard" class="page">
  <div class="container">
    <!-- light-theme cards, stats, charts -->
  </div>
</div>

<!-- Agent HQ page (hidden by default) -->
<div id="pageHQ" class="page" style="display:none">
  <div style="background:#0f0f23;min-height:100vh">
    <h1 style="font-family:'Press Start 2P',monospace;color:#ffd700">🏢 AGENT HQ</h1>
    <div id="hqCanvasContainer" style="min-height:400px"></div>
    <div id="hqAgentGrid"></div>
  </div>
</div>
```

### Tab Switching (key: lazy canvas init)

```javascript
let currentTab = 'dashboard';

function switchTab(tab) {
  currentTab = tab;
  
  if (tab === 'dashboard') {
    document.getElementById('pageDashboard').style.display = 'block';
    document.getElementById('pageHQ').style.display = 'none';
    document.getElementById('tabDashboard').style.background = '#10b981';
    document.getElementById('tabHQ').style.background = '#6b7280';
  } else {
    document.getElementById('pageDashboard').style.display = 'none';
    document.getElementById('pageHQ').style.display = 'block';
    document.getElementById('tabDashboard').style.background = '#6b7280';
    document.getElementById('tabHQ').style.background = '#ffd700';
    
    // LAZY: Only create canvas when first entering this tab
    if (!window.hqInitialized) {
      initAgentHQ();
      window.hqInitialized = true;
    }
  }
}
```

### Agent Status Grid (real API data)

```javascript
setInterval(async () => {
  if (currentTab === 'hq') {
    try {
      const agentsData = await api('/api/agents');
      renderHQAgents(agentsData.agents);
    } catch (e) { console.error(e); }
  }
}, 15000);
```

## Why This Works

1. **Dashboard page** — light theme loads immediately, no canvas overhead.
2. **Agent HQ page** — canvas in `display:none` — wastes zero resources.
3. **First tab switch** — creates canvas when container is visible, correct size.
4. **Subsequent switches** — canvas already exists, just toggles visibility.

## Pitfalls

- **Never pre-create canvas in a `display:none` container** — canvas width/height will be 0, all drawing invisible. Create it on first tab switch via `document.createElement`.
- **Header overlap**: hide the main header when switching to Agent HQ for full immersion: `document.querySelector('.header').style.display = 'none'`.

## When to Use

- User wants both a **clean data dashboard** AND a **pixel art agent visualization** as separate tab pages.
- User explicitly rejected pixel art for the main dashboard but wants it as a secondary view.