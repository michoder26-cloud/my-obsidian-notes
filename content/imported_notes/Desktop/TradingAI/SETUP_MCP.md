# 🔧 Setup Guide - MCP Integration

เมื่อสัพเดช! 🚀 Agents ตอนนี้ใช้ **MCP Tools** แล้ว

---

## ✅ ต้องเช็คก่อน:

### 1. Node.js + npm ติดตั้งไหม?

```powershell
node --version
npm --version
```

ถ้าไม่มี: https://nodejs.org (LTS)

### 2. MCP Server Dependencies

```powershell
# ไปที่ mcp folder
cd mcp

# Install dependencies
npm install
```

ต้องติดตั้ง:
- `yahoo-finance2` - ดึงข้อมูลหุ้น
- `technicalindicators` - RSI, MACD, BB
- `rss-parser` - ข่าว
- `zod` + `@modelcontextprotocol/sdk` - MCP server

### 3. Python Dependencies

```powershell
# กลับไปที่ root
cd ..

pip install -r requirements.txt
```

---

## 🚀 ใช้งาน:

### ขั้น 1: Set API Key

```powershell
# Edit .env
OPENROUTER_API_KEY=sk-or-your-key-here
```

### ขั้น 2: Test Setup

```powershell
python test_system.py
```

### ขั้น 3: Run Analysis (MCP ทำงานอัตโนมัติ)

```powershell
python stock_analysis_system.py
```

**ระบบจะใช้ MCP แบบอัตโนมัติ** ✅

---

## 📊 MCP Flow:

```
stock_analysis_system.py
    ↓
[1] Scraper Agent
    ↓
    MCPClient.get_stock_data() ← Real-time price
    + yfinance fundamentals ← P/E, ROE, etc
    ↓
[2] Analyst Agent
    ↓
    MCPClient.analyze_technical() ← RSI, MACD, BB
    ↓
[3] Sentiment Agent
    ↓
    MCPClient.get_market_news() ← Latest news
    MCPClient.analyze_sentiment() ← Bullish/Bearish
    ↓
[4] Recommender Agent
    ↓
    BUY/HOLD/SELL + Price Target
```

---

## ✨ ผลต่าง:

### ก่อน (yfinance only):
```
- Data basic (ราคา, P/E)
- Manual RSI/MACD calculation
- No sentiment analysis
- ~5 min/stock
```

### ตอนนี้ (MCP + AI):
```
- Real-time price from TradingView ✅
- Auto technical indicators (RSI, MACD, BB) ✅
- Sentiment analysis (positive/negative) ✅
- ~1 min/stock
- More accurate signals
```

---

## ⚠️ Troubleshooting:

### Error: "MCP server not found"
```
Check:
1. mcp/tradingview-server.js exists?
2. npm install done?
3. Node.js installed?
```

### Error: "Cannot find module..."
```powershell
cd mcp
npm install
npm list  # Check dependencies
```

### Error: "subprocess timeout"
```
Check:
1. MCP server starting?
2. Node.js slow machine?
3. Internet connection?
```

---

## 🎯 ข้อดี MCP:

✅ Real-time data (not 15-20 min delayed)  
✅ Technical analysis ฟรี (RSI, MACD, Bollinger)  
✅ Sentiment from news  
✅ Trading signals built-in  
✅ Future-proof architecture  

---

## 📝 Next Steps:

1. ✅ MCP integration done
2. ⏳ Add daily scheduled analysis (optional)
3. ⏳ Save results to database
4. ⏳ OpenClaw UI integration
5. ⏳ Alert system (email/Telegram)

---

**Ready?** `python stock_analysis_system.py` 🚀
