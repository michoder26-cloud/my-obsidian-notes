# 🤖 Doraemon AI: Self-Learning 24-Hour Autonomous Trader

**XAU/USD Technical Analysis Agent**  
**Status**: ✅ Ready to Run  
**Version**: 1.0  
**Last Updated**: 2026-05-16

---

## 📋 System Overview

Doraemon AI is an autonomous trading agent that:
- ✅ Analyzes XAU/USD every **15 minutes**
- ✅ Generates trading signals (BUY/SELL/HOLD)
- ✅ Reports signals to **Discord**
- ✅ **Never forgets** - Maintains persistent learning memory
- ✅ Gets **smarter** with every trade

### Key Features:
```
┌─────────────────────────────────────┐
│  Every 15 Minutes:                  │
├─────────────────────────────────────┤
│  1. Fetch XAU/USD data             │
│  2. Calculate indicators            │
│  3. Generate signal                 │
│  4. Report to Discord 📢           │
│  5. Update memory 💾               │
│  6. Learn & improve 🧠             │
└─────────────────────────────────────┘
```

---

## 📁 Project Structure

```
C:\Users\TKTF\Desktop\doraemon\
├── 🤖 doraemon_agent.py          # Main analysis engine
├── ⏰ run_doraemon.py            # Scheduler (15-min interval)
├── 📝 trading_rules.md           # Rules (Doraemon must follow)
├── 📊 trading_memory.md          # Memory (Learning journal)
├── .env                          # Configuration (API keys)
├── requirements.txt              # Python dependencies
└── doraemon_trading.log          # Auto-generated log file
```

---

## 🚀 Quick Start

### 1. Install Dependencies

```bash
cd C:\Users\TKTF\Desktop\doraemon
pip install -r requirements.txt
```

### 2. Update .env File

Edit `.env` and fill in:
```
MT5_USERNAME=your_username       (can wait until Monday)
MT5_PASSWORD=your_password       (can wait until Monday)
MT5_SERVER=broker_server         (can wait until Monday)
DISCORD_WEBHOOK_URL=...          (✅ Already updated)
```

### 3. Run Doraemon

**Option A: Single Analysis (test)**
```bash
python doraemon_agent.py
```

**Option B: Continuous (every 15 min)**
```bash
python run_doraemon.py
```

---

## 📊 How It Works

### Flow Diagram:
```
Start
  ↓
┌─────────────────────────────┐
│ 1. READ trading_rules.md    │ ← Strategy rules
│    (What to do)              │
├─────────────────────────────┤
│ 2. FETCH XAU/USD Data       │ ← Alpha Vantage API
├─────────────────────────────┤
│ 3. ANALYZE                  │
│    - Calculate RSI           │
│    - Calculate MACD          │
│    - Calculate EMA           │
│    - Calculate Bollinger Bands
├─────────────────────────────┤
│ 4. GENERATE SIGNAL          │
│    BUY / SELL / HOLD        │
│    + Confidence %            │
├─────────────────────────────┤
│ 5. SEND DISCORD REPORT 📢  │
├─────────────────────────────┤
│ 6. UPDATE MEMORY 💾        │
│    (learning_memory.md)      │
└─────────────────────────────┘
  ↓
Wait 15 Minutes
  ↓
Repeat (Loop forever)
```

---

## 🧠 Memory System (Never Forgets!)

### Two Memory Files:

**File 1: `trading_rules.md`**
- Trading strategy & rules
- Risk management settings
- Signal criteria
- Doraemon reads this **BEFORE** every trade

**File 2: `trading_memory.md`**
- Learning journal
- Trade history
- Performance statistics
- Lessons learned
- **Updated after every analysis**

### How Learning Works:
```
Analysis Cycle:
  1. Read trading_rules.md (get instructions)
  2. Analyze current market
  3. Generate signal
  4. Add result to trading_memory.md
  5. Next cycle reads updated memory
  → Gets smarter each time! ✓
```

---

## 📢 Discord Integration

Doraemon sends signal reports to Discord with:
- 🟢 BUY Signal (Green)
- 🔴 SELL Signal (Red)
- 🟡 HOLD Signal (Yellow)
- Current price
- Confidence level
- Technical indicators
- Analysis reasons

**Example Discord Message:**
```
🤖 Doraemon AI - BUY Signal

Price: $2,450.50
Confidence: 85.0%
RSI (14): 28.3
MACD: 0.000145
EMA Ratio: 1.25%

Reasons:
- RSI 28.3 < 30 (Oversold)
- MACD crosses above Signal Line
- EMA 12 > EMA 26 (Uptrend)
```

---

## ⚙️ Configuration

### Environment Variables (.env):

| Variable | Purpose | Current |
|----------|---------|---------|
| ALPHA_VANTAGE_API_KEY | Fetch XAU/USD data | ✅ |
| DISCORD_WEBHOOK_URL | Send reports | ✅ |
| MT5_USERNAME | Trading platform | ⏳ Monday |
| MT5_PASSWORD | Trading platform | ⏳ Monday |
| MT5_SERVER | Trading broker | ⏳ Monday |
| INITIAL_BALANCE | Account size | $10,000 |
| RISK_PERCENT | Risk per trade | 10% |
| MIN_CONFIDENCE | Min signal strength | 40% |

---

## 🎯 Trading Rules Summary

### Entry Rules:
```
BUY When:
  ✓ RSI < 30 (Oversold)
  ✓ MACD crosses above
  ✓ EMA 12 > EMA 26
  ✓ Confidence > 40%

SELL When:
  ✓ RSI > 70 (Overbought)
  ✓ MACD crosses below
  ✓ EMA 12 < EMA 26
  ✓ Confidence > 40%

HOLD When:
  ✓ Conflicting signals
  ✓ Confidence < 40%
```

### Risk Management:
```
- Risk: 10% of account per trade
- Max Positions: 2 concurrent
- Max Daily Loss: 5%
- Max Drawdown: 10%
- Stop Loss: 50 pips max
```

---

## 📊 Logging & Monitoring

All activity logged to `doraemon_trading.log`:
- Analysis timestamps
- Signal details
- Discord reports
- Errors & warnings
- Memory updates

View log:
```bash
tail -f doraemon_trading.log
```

---

## 🔄 Continuous Learning Process

### Every 15 Minutes:
```
✓ Analyze market
✓ Generate signal
✓ Report to Discord
✓ Update memory (learning)
```

### Every 24 Hours:
```
✓ Review all trades
✓ Calculate win rate
✓ Identify patterns
✓ Update strategy if needed
```

### Every 7 Days:
```
✓ Full performance review
✓ Backtest new ideas
✓ Optimize parameters
✓ Reset if performance < 50%
```

---

## 🚨 Emergency Controls

If something goes wrong:

**Stop Doraemon:**
```bash
Ctrl + C
```

**Reset Memory:**
```bash
rm trading_memory.md
# (Creates new journal next run)
```

**View Status:**
```bash
tail -100 doraemon_trading.log
```

---

## 📈 Performance Targets

- **Win Rate**: > 55%
- **Profit Factor**: > 1.5
- **Monthly Return**: > 5%
- **Max Drawdown**: < 10%

---

## 🔧 Next Steps (Monday)

### Add MT5 Trading:
1. Get MT5 login credentials
2. Update .env file (MT5_USERNAME, MT5_PASSWORD, MT5_SERVER)
3. Integrate MT5 sub-agent
4. Execute actual trades (not just signals)

### Current Status (Weekend):
- ✅ Signal generation works
- ✅ Discord reporting works
- ✅ Memory system works
- ⏳ MT5 execution (Monday)

---

## 📞 Support

If Doraemon crashes:
1. Check `.env` configuration
2. Verify Discord webhook URL
3. Check internet connection
4. Review `doraemon_trading.log`
5. Restart: `python run_doraemon.py`

---

## ⭐ Golden Rule

> **Before every decision, Doraemon reads:**
> 1. trading_rules.md (What to do)
> 2. trading_memory.md (What I learned)
>
> **This ensures:**
> - Consistent strategy
> - Continuous learning
> - Never forgets anything
> - Gets smarter every trade

---

**Status**: 🟢 Ready to Trade  
**Last Update**: 2026-05-16  
**Version**: 1.0  
**Developer**: Claude Code 🤖

