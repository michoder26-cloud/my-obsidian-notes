---
name: self-learning-trading-loop
description: >-
  Add a self-improvement feedback loop to any automated trading bot. SQLite trade
  DB → post-trade feature analysis → parameter auto-tuning → enhanced agent prompts.
  Every trade makes the bot smarter.
category: trading
related_skills:
  - claw-trade-mt5-linux
  - claw-trade-linux-migration
---

# Self-Learning Feedback Loop for Trading Bots

## When to Use

The bot runs autonomously but shows **no improvement over time** — same win rate week after week, static parameters, Discord/admin reports that don't drive change. The bot needs a learning system that **remembers every trade, analyzes what works, and adjusts itself.**

This skill covers the pattern. It was built for Claw_Trade (XAU/USD, FBS, MT5 + mt5linux) but the architecture is broker-agnostic.

## Architecture

```
EVERY TRADE CYCLE:

┌────────────┐    ┌──────────────────┐    ┌───────────────┐
│  Trade     │───►│  SQLite DB       │───►│  Feature      │
│  Opens     │    │  (full context)  │    │  Win Rates    │
└────────────┘    └──────────────────┘    └───────┬───────┘
                                                   │
                                                   ▼
┌────────────┐    ┌──────────────────┐    ┌───────────────┐
│  CEO       │◄───┤  Enhanced        │◄───┤  Parameter    │
│  Decision  │    │  Prompt + Stats  │    │  Optimizer    │
└────────────┘    └──────────────────┘    └───────────────┘
                                                   │
                                                   ▼
                                          ┌───────────────┐
                                          │  Learned      │
                                          │  Config       │
                                          │  (JSON file)  │
                                          └───────────────┘

WEEKLY:

┌──────────────┐    ┌──────────────────┐    ┌───────────────┐
│  Saturday    │───►│  Learning        │───►│  Discord      │
│  10:00 UTC   │    │  Engine Report   │    │  Webhook      │
└──────────────┘    └──────────────────┘    └───────────────┘
  (Dynamic stats, trend, param changes, lessons)
```

## Components

### 1. `learning_engine.py` — The Core

Three classes in one module:

| Class | Responsibility |
|-------|---------------|
| `TradeDatabase` | SQLite: store every trade with full market context (regime, session, fibo_zone, macd_state, confidence) |
| `LearningOptimizer` | Analyze trades → adjust parameters (confidence thresholds, position sizing, SL multiplier, session bias) |
| `patch_discord_reporter()` | Monkey-patches `DiscordReporter` with `report_learning_summary()` for dynamic reflection |

### 2. SQLite Schema

```sql
-- Every trade with context features
CREATE TABLE trades (
    ticket INTEGER UNIQUE,
    signal TEXT, entry_price REAL, exit_price REAL,
    pnl_usd REAL, close_reason TEXT, r_achieved REAL,
    regime TEXT, session TEXT, fibo_zone TEXT,
    macd_state TEXT, confidence REAL, lot_multiplier REAL,
    entry_time TEXT, exit_time TEXT, lesson_learned TEXT
);

-- Running win rates by feature (for fast lookup)
CREATE TABLE feature_winrates (
    feature_name TEXT, feature_value TEXT,
    total_trades INTEGER, wins INTEGER, winrate REAL,
    UNIQUE(feature_name, feature_value)
);

-- Audit trail for parameter changes
CREATE TABLE parameter_history (
    param_name TEXT, old_value REAL, new_value REAL, reason TEXT
);

CREATE TABLE weekly_stats (
    week_number INTEGER, year INTEGER,
    total_trades INTEGER, wins INTEGER, losses INTEGER,
    total_pnl REAL, best_regime TEXT, worst_regime TEXT
);
```

### 3. Features Tracked

These are the **dimensions** for analysis — each trade record carries all of them so the optimizer can slice by any dimension:

| Feature | Values | Why it matters |
|---------|--------|--------------|
| **regime** | TRENDING, RANGING, HIGH_VOLATILITY, LOW_LIQUIDITY | Different regimes demand different thresholds |
| **session** | TOKYO_LONDON, LONDON, NY, NY_LATE, ASIA | Some sessions may have better setups |
| **fibo_zone** | equilibrium, discount_premium, all_in_market_maker, neutral | Market structure zones affect success rate |
| **macd_state** | bullish_cross, bearish_cross, neutral | Momentum confirmation quality |
| **confidence_bucket** | ULTRA_HIGH(95%+), HIGH(90%+), MODERATE_HIGH(85%+), MIN_THRESHOLD(78%+), LOW | Calibrate confidence vs actual results |

### 4. Parameter Auto-Tuning Rules

These fire **immediately after each trade close** (not batch/periodic):

| Rule | Trigger | Action |
|------|---------|--------|
| Regime WR < 35% | ≥5 trade samples in regime | Raise confidence threshold +0.04 for that regime |
| Regime WR > 65% | Threshold > 0.75 | Lower threshold -0.02 for that regime |
| Session WR < 30% | ≥3 trade samples | Reduce session_multiplier -0.15 |
| Session WR > 60% | Multiplier < 1.2 | Increase session_multiplier +0.10 |
| Negative expectancy | ≥5 trade samples in regime | Reduce position_size_pct -1% for that regime |
| Positive expectancy > 0.5 | ≥5 trade samples | Increase position_size_pct +1% for that regime |
| Avg R:R < target × 0.7 | ≥10 closed trades | Widen SL by 15% (sl_atr_multiplier × 1.15) |
| Consecutive losses ≥ 3 | Always | Reduce ALL position sizes by 30% (auto-cool-off) |

### 5. Integration Points

The orchestrator needs **three integration points**:

```python
# A) Record entry (when trade opens)
get_db().record_entry({
    "ticket": ticket,
    "signal": decision,
    "entry_price": price,
    "regime": regime,
    "fibo_zone": fibo_zone,
    "macd_state": macd_cross,
    "confidence": confidence,
    ...
})

# B) Record exit + run optimizer (when trade closes)
get_db().record_exit(ticket, exit_data)
run_optimization()  # Auto-saves learned_config.json

# C) Use learned params instead of hardcoded
learned_threshold = opt.params["confidence_threshold"].get(regime, 0.78)
learned_pct = opt.params["position_size_pct"].get(regime, 6.0)

# D) Inject learning stats into CEO prompt
enhanced_memory = [opt.get_enhanced_ceo_context()] + raw_learning_memory
ceo_agent.decide(..., enhanced_memory)
```

### 6. CEO Prompt Enhancement

The `get_enhanced_ceo_context()` returns a string like:

```
📊 **LEARNING ENGINE — LIVE STATS:**
📈 Overall Win Rate: 62.5%
⚠️ Consecutive Losses: 2
🏷️ Win Rate in TRENDING: 80%
🏷️ Win Rate in RANGING: 40%
🕐 Win Rate in TOKYO_LONDON: 75%
🕐 Win Rate in NY: 50%

⚙️ **Current Parameter Overrides:**
  - RANGING: confidence >= 82%
  - SL Multiplier: 1.15x ATR
  - R:R Target: 3.0
```

This prepended to the CEO agent's learning_memory input makes the AI aware of its recent performance.

### 7. Weekly Reflection (Dynamic)

The old static reflection becomes dynamic:

```
📊 รายงานทบทวนการเทรดทองคำ (XAU/USD Weekly Reflection)

📈 ผลงานรอบ 7 วันที่ผ่านมา:
├─ ออเดอร์ทั้งหมด: 12 ไม้
├─ ชนะ: 7 | แพ้: 5
├─ Win Rate: 58.3%
├─ กำไร/ขาดทุนรวม: $+245.00

🔍 Win Rate Breakdown:
├─ regime:
│  🟢 TRENDING: 80% (กำลังดี)
│  🔴 RANGING: 33% (ต้องปรับปรุง)
├─ session:
│  🟢 TOKYO_LONDON: 75% (กำลังดี)
│  🔴 NY: 33% (ต้องปรับปรุง)

⚙️ การปรับพารามิเตอร์อัตโนมัติ:
├─ RANGING: เพิ่มความมั่นใจขั้นต่ำเป็น 84%
├─ NY: ลดน้ำหนักเป็น 0.55x
└─ Avg R:R achieved = 2.1 (target: 3.0)

⚠️ คำเตือน: ติดลบติดต่อกัน 4 ครั้งแล้ว — ลด Lot ลง 30%
```

## Files to Create

| File | Contents |
|------|----------|
| `src/learning_engine.py` | Full module (TradeDatabase + LearningOptimizer + helpers) |
| `learned_config.json` | Initial parameter defaults (auto-generated on first optimization) |

## Files to Patch

| File | What to change |
|------|---------------|
| `src/orchestrator.py` | (A) Import learning_engine; (B) Record entry after trade opens; (C) Record exit + run_optimization() after trade closes; (D) Use learned confidence_threshold per regime; (E) Use learned position_size_pct per regime; (F) Inject learning context into CEO decision call |
| `src/agents.py` | Replace `TradeReflectionEngine.run_weekly_reflection()` — use `get_optimizer().get_reflection_report()` instead of static template. Add fallback `_static_fallback()`. |

## Pitfalls

- **SQLite threading**: TradeDatabase opens/closes connections per call (no persistent connection) — avoids thread-safety issues in multi-agent orchestrators that call from different threads.
- **Win rate calculation**: Feature winrates update on INSERT only (after trade close). The `ON CONFLICT ... DO UPDATE SET` properly handles cumulative stats. The `wins = CASE WHEN ? THEN wins+1 ELSE wins END` pattern works because Python bools serialize to SQLite 0/1.
- **Cold start**: The optimizer needs ≥3 trades per feature to make adjustments. First few trades use defaults from `learned_config.json`. No old-history migration needed — DB starts empty.
- **Consecutive losses trigger**: When ≥3 in a row, all position sizes drop 30% ACROSS ALL regimes. This is intentional — a losing streak suggests a systematic issue, not a regime-specific one.
- **The `_classify_session` method** uses `datetime.fromisoformat()` not `pd.to_datetime()` — avoids pandas dependency in the learning module.
- **`learned_config.json` path**: The default path is `{repo_root}/learned_config.json`. If the orchestrator runs from a different cwd, pass an absolute config_path to `LearningOptimizer`.
- **Don't overwrite CEO prompt injection**: The enhanced context string is prepended to the learning_memory list — don't replace the existing list, which may carry other lessons. The CEO agent already handles `learning_memory[-5:]` internally.
- **Discord webhook fallback**: Weekly reflection tries `DISCORD_WEEKLY_REFLECTION_WEBHOOK` first, falls back to `DISCORD_GOLD_REFLECTION_WEBHOOK`. Both are optional — no crash if unset.