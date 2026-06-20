# Doraemon Trading Rules & Strategy
**Last Updated**: 2026-05-16  
**Version**: 1.0

---

## 📋 CORE TRADING RULES (DORAEMON MUST FOLLOW)

### 1. INSTRUMENT & TIMEFRAME
- **Pair**: XAU/USD only
- **Timeframe**: Every 15 minutes analysis
- **Data Source**: Alpha Vantage API + Market data
- **Broker**: MT5

### 2. POSITION SIZING (10% Risk Rule)
```
Rule: Risk = 10% of account balance per trade
Example:
  - Account: $10,000
  - Risk per trade: $1,000 (10%)
  - Position size: Calculate based on SL distance
```

- **Calculation**:
  ```
  Position Size = (Account Balance × Risk %) / (Entry - Stop Loss)
  ```
- **Max Concurrent Positions**: 2
- **Max Daily Trades**: 10

### 3. ENTRY SIGNALS (Technical Indicators)

#### Signal Requirements:
```
BUY Signal When:
  ✓ RSI (14) < 30 (Oversold)
  ✓ MACD crosses above Signal Line
  ✓ EMA 12 > EMA 26 (Uptrend)
  ✓ Price inside Bollinger Bands (Low side)
  Confidence: Count matching conditions × 20%

SELL Signal When:
  ✓ RSI (14) > 70 (Overbought)
  ✓ MACD crosses below Signal Line
  ✓ EMA 12 < EMA 26 (Downtrend)
  ✓ Price inside Bollinger Bands (High side)
  Confidence: Count matching conditions × 20%

HOLD/NO TRADE When:
  ✓ Conflicting signals
  ✓ Confidence < 40%
  ✓ News event expected
```

### 4. EXIT RULES (Stop Loss & Take Profit)

#### Stop Loss (Mandatory):
- **SL Distance**: 50 pips maximum
- **SL Type**: Hard stop at calculated level
- **Placement**: Just below support (BUY) or resistance (SELL)

#### Take Profit:
- **TP Target**: Risk:Reward = 1:1.5 minimum
- **Trail SL**: Move SL to breakeven when +20 pips profit
- **Partial Profits**: Close 50% at 1:1.5 RR, let 50% run

#### Force Exit Conditions:
- Daily loss > 5% of account
- Drawdown > 10%
- News event within 30 minutes
- Price action rejection

### 5. RISK MANAGEMENT (CRITICAL)

```
Account Protection:
  Max Daily Loss: 5% of account
  Max Drawdown: 10% of account
  Max Open Positions: 2
  Max Position Size: 10% risk per trade

Trade Rejection Rules:
  - Reject if SL > 100 pips
  - Reject if spread > 5 pips
  - Reject if news event in 1 hour
  - Reject if confidence < 40%
```

### 6. ANALYSIS CHECKLIST (BEFORE EVERY TRADE)

Before executing ANY trade, Doraemon MUST check:

- [ ] Read this file (trading_rules.md) ✓
- [ ] Current account balance
- [ ] Open positions count
- [ ] Daily loss amount
- [ ] Technical indicators (RSI, MACD, EMA, BB)
- [ ] Confidence level calculation
- [ ] SL/TP levels validation
- [ ] Position size calculation
- [ ] News calendar check
- [ ] Update trading_memory.md

### 7. LEARNING & IMPROVEMENT

**After each trade (WIN or LOSS):**
1. Document in trading_memory.md:
   - Entry reason & signal strength
   - Exit reason & profit/loss
   - What worked / What didn't
   - Improvement for next time

2. Review every 24 hours:
   - Trading pattern analysis
   - Win rate calculation
   - Average profit/loss per trade
   - Common mistakes
   - Lessons learned

3. Update strategy if:
   - Win rate < 50% (2 days rolling)
   - Drawdown > 10%
   - Same mistake repeated 3× times

---

## ⚠️ EMERGENCY RULES (NEVER BREAK THESE)

1. **STOP ALL TRADING IF:**
   - Daily loss reaches 5%
   - Drawdown exceeds 10%
   - Critical news event occurs
   - MT5 connection fails
   - Account balance < $1,000

2. **IMMEDIATE ACTIONS:**
   - Close all losing positions
   - Alert to Discord immediately
   - Update trading_memory.md
   - Log error details
   - Wait for manual approval

---

## 📊 PERFORMANCE TARGETS

- **Win Rate**: Target > 55%
- **Average Win**: > Average Loss
- **Profit Factor**: > 1.5
- **Monthly Return**: Target > 5%

---

## 🔄 CONTINUOUS IMPROVEMENT PROCESS

### Every Trade Cycle (15 min):
1. Analyze market
2. Generate signal
3. Check rules ✓
4. Execute trade (if qualified)
5. Monitor position
6. Update memory

### Every 24 Hours:
1. Review all trades
2. Calculate metrics
3. Identify patterns
4. Learn from mistakes
5. Update strategy if needed

### Every 7 Days:
1. Full strategy review
2. Backtest new ideas
3. Optimize parameters
4. Update trading_rules.md
5. Reset if < 50% win rate

---

## 📝 REVISION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-05-16 | Initial rules creation |
| | | |

---

**⭐ GOLDEN RULE:** 
> Doraemon reads this file BEFORE every trade decision.
> This ensures consistent strategy and prevents emotional trading.
> When in doubt, HOLD and re-analyze!



## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%

## Stop Loss Enhancement
STOP_LOSS_PERCENT=3.0  # Exit if loss exceeds 3%