# Professional Dark Theme Trading Dashboard

## When to use

- User wants a **professional trading dashboard** (not pixel art, not light corporate)
- User wants **realtime live price** from MT5 via mt5linux
- User wants **equity curve**, **regime breakdown**, **monthly P&L** charts
- User says things like "ให้ดูเป็นมืออาชีพ", "realtime จากบอทเทรด", "กราฟการเติบโต"

## Color Palette

```css
:root {
  --bg: #0a0a0f;
  --bg2: #111118;
  --card: #16161f;
  --border: #2a2a3a;
  --text: #e8e8f0;
  --text2: #9898b0;
  --text3: #5a5a78;
  --green: #00e676;
  --red: #ff5252;
  --gold: #ffd700;
  --blue: #448aff;
  --purple: #b388ff;
  --cyan: #18ffff;
}
```

## Layout

Topbar → Live Ticker → Regime Bar → Metric Cards → Charts Row (equity + doughnut) → Bottom (monthly bars + recent trades table)

## Live Price API

```python
try:
    from mt5linux import MetaTrader5 as _MT5
except ImportError:
    _MT5 = None

def get_live_price(self) -> dict:
    if _MT5 is None:
        return {"ok": False, "price": 0}
    try:
        m = _MT5(host='localhost', port=8001)
        m.initialize()
        tick = m.symbol_info_tick("XAUUSDc")
        if tick:
            return {"price": round(tick.ask, 2), "bid": round(tick.bid, 2),
                    "ask": round(tick.ask, 2), "spread": round((tick.ask - tick.bid) * 100, 1),
                    "change": round(tick.ask - tick.last, 2) if tick.last else 0, "ok": True}
    except Exception:
        pass
    return {"ok": False, "price": 0}
```

## Charts (Chart.js 4.4 CDN)

- Equity curve: line, gold, gradient fill, tension 0.4, no points
- Regime: doughnut, cutout 65%, 5 colors
- Monthly P&L: bar, green/red, borderRadius 4

## Pitfalls

1. mt5linux import: always try/except at module level
2. Chart.js leak: call chart.destroy() before re-creating
3. CORS: add Access-Control-Allow-Origin: * to all API responses
4. Monospace (JetBrains Mono) for prices/P&L/timestamps
