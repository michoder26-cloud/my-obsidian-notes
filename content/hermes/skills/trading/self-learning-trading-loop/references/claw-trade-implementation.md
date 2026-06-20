# Claw_Trade Self-Learning Implementation (June 2026)

## Environment

- **Repo**: `/root/Claw_Trade` (https://github.com/michoder26-cloud/Claw_Trade)
- **Host**: AWS EC2 (13.140.183.183) — AMD EPYC, 6-core, 11GB RAM
- **OS**: Ubuntu 24.04, Python 3.12
- **Broker**: FBS (Demo: FBSTradestone-Demo, $10,450.96)
- **Symbol**: XAUUSDc (gold)
- **Bot**: MMTC v2.0 5-Agent (Quant, News, Bull, Bear, CEO) via OpenRouter
- **MT5**: Docker container `claw-trade-mt5` (gmag11/metatrader5_vnc) + mt5linux port 8001

## Files Created

### `src/learning_engine.py` — 34KB, ~400 lines

Three classes + 4 module-level functions. Full code at the path above.

### `learned_config.json`

Default parameters before any optimization:

```json
{
  "confidence_threshold": {
    "TRENDING": 0.78, "RANGING": 0.82,
    "HIGH_VOLATILITY": 0.85, "LOW_LIQUIDITY": 0.90
  },
  "position_size_pct": {
    "TRENDING": 6.0, "RANGING": 4.0,
    "HIGH_VOLATILITY": 3.0, "LOW_LIQUIDITY": 2.0
  },
  "sl_atr_multiplier": 1.0,
  "rr_ratio": 3.0,
  "session_multiplier": {
    "TOKYO_LONDON": 1.0, "NY": 1.0,
    "NY_LATE": 0.7, "ASIA": 0.5
  },
  "fibo_zone_bias": {
    "equilibrium": 1.0, "discount_premium": 1.2,
    "all_in_market_maker": 1.5, "neutral": 0.8
  }
}
```

## Files Patched

### `src/orchestrator.py` — 5 patches

1. **Line 17**: `from learning_engine import get_db, get_optimizer, run_optimization`
2. **Lines 580-584**: Enhanced CEO memory — prepends `opt.get_enhanced_ceo_context()` to learning_memory
3. **Lines 598-601**: Uses `learned_threshold = opt.params["confidence_threshold"].get(regime, 0.78)` instead of hardcoded 0.78
4. **Lines 689-703**: Uses `learned_pct = opt.params["position_size_pct"].get(regime, 6.0)` instead of hardcoded POSITION_SIZE_PERCENT
5. **Lines 757-783**: After trade open, records entry via `get_db().record_entry({...})`
6. **Lines 1245-1227**: After trade close, records exit via `get_db().record_exit(ticket, {...})` + `run_optimization()`

### `src/agents.py` — Replaced `TradeReflectionEngine.run_weekly_reflection()`

Now calls `get_optimizer().get_reflection_report()` for dynamic report. Old static code preserved as `_static_fallback()`.

## Discord Webhooks

The user's `.env` has several Discord webhooks. Key ones for learning:

- `DISCORD_WEEKLY_REFLECTION_WEBHOOK` — weekly learning report
- `DISCORD_GOLD_REFLECTION_WEBHOOK` — fallback
- `DISCORD_TRADE_WEBHOOK_URL` — per-trade open/close notifications

## Test Results (all passed)

- Database creation: ✅
- Record entry + exit: ✅
- Feature win rates calculated correctly: ✅
- Parameter optimization (confidence thresholds, position sizing): ✅
- Reflection report generated: ✅
- CEO context with learning stats: ✅
- All imports (`src/orchestrator`, `src/agents`, `src/learning_engine`, `main.py`): ✅