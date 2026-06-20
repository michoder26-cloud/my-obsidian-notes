# 🚀 Stock Analysis System - LIVE with MCP Tools

**Real-time analysis using Claude + MCP trading tools**

---

## ✅ ก่อนรัน: ต้องมี Node.js

```powershell
# Check Node.js + npm installed
node --version
npm --version
```

ถ้าไม่มี: https://nodejs.org (LTS)

---

## 🔧 Setup MCP Tools (ทำแค่ครั้งเดียว)

```powershell
# 1. ไปที่ mcp folder
cd C:\Users\TKTF\Desktop\TradingAI\mcp

# 2. Install MCP dependencies
npm install

# 3. Back to root
cd ..
```

**✓ Done!**

---

## 🎬 Setup Python

```powershell
# Install Python dependencies
pip install -r requirements.txt

# Set API Key in .env
# OPENROUTER_API_KEY=sk-or-your-key
```

---

## 🚀 RUN!

```powershell
python stock_analysis_system.py
```

---

## 📊 ระบบจะทำอะไร:

### Flow:
```
[1] Scraper Agent
    → Claude uses MCP get_stock_data()
    → Real-time price + fundamentals
    ↓

[2] Analyst Agent  
    → Claude uses MCP analyze_technical()
    → RSI, MACD, Bollinger Bands
    ↓

[3] Sentiment Agent
    → Claude uses MCP get_market_news()
    → Claude uses MCP analyze_sentiment()
    → News + sentiment analysis
    ↓

[4] Recommender Agent
    → Claude uses MCP generate_trading_signal()
    → Combines all data
    → BUY/HOLD/SELL + Price Target
```

---

## 📈 ผลลัพธ์:

```
🎯 Recommendation: BUY
   Confidence: 87%
   Price Target: $220 (in 12 months)
   Upside: 12%
   Downside Risk: 5%

📌 Key Reasons:
   • Undervalued P/E ratio
   • Strong revenue growth
   • Positive analyst sentiment

⚠️ Risks:
   • Market volatility
   • Rising interest rates
   • Competition from Google

🚪 Exit Conditions:
   • Take profit at $220
   • Stop loss at $190
```

---

## 🔄 Real-time Data:

✅ **Claude → MCP tools → Real-time data**

Data sources:
- **Prices**: Yahoo Finance (via MCP)
- **Technical**: Calculated live by MCP
- **News**: Yahoo Finance RSS (via MCP)
- **Signals**: MCP momentum analysis

**NOT delayed** - Claude calls MCP tools ตอนรัน!

---

## 📌 เปลี่ยนหุ้น:

Edit `stock_analysis_system.py`:

```python
stocks = ["GOOGL"]  # ← เปลี่ยนเป็น AAPL, MSFT, etc.

for symbol in stocks[:1]:  # [:1]=1หุ้น, [:5]=5หุ้น
    orchestrator.analyze_stock(symbol)
```

---

## ❓ ติดขัด?

### "Cannot find module"
```powershell
cd mcp
npm install
npm list  # Check what's installed
```

### "MCP tool timeout"
```
Check:
1. Node.js running slow?
2. Internet connection ok?
3. Try again
```

### "API Error: 401"
```
Check:
1. .env file has OPENROUTER_API_KEY
2. API key is correct
3. Account has credits (openrouter.ai)
```

---

## 💡 Flow Details:

1. **Python Orchestrator** sends prompt to OpenRouter
2. **OpenRouter API** receives prompt + MCP tools list
3. **Claude** decides which tools to use
4. **MCP Server** (Node.js) executes tools
5. **Claude** gets results back
6. **Claude** synthesizes and gives recommendation
7. **Python** displays results

**Everything real-time!** ⚡

---

**Ready?** 👉 `python stock_analysis_system.py`

---

**Note**: First run may be slow (Node modules loading). Subsequent runs faster.
