# 🚀 AI Trading Setup Guide
## Claude Code + TradingView MCP

### ✅ ขั้นตอนการติดตั้ง (Installation Steps)

#### **ขั้นตอนที่ 1: เตรียมสภาพแวดล้อม**
```powershell
# เปิด PowerShell หรือ Terminal
# ไปที่โฟลเดอร์โปรเจกต์
cd C:\Users\TKTF\Desktop\TradingAI

# เช็คว่า Node.js ติดตั้งแล้วหรือไม่
node --version
npm --version
```

#### **ขั้นตอนที่ 2: ติดตั้ง Dependencies**
```powershell
npm install
```

#### **ขั้นตอนที่ 3: เปิดใช้งาน Claude Code**
```powershell
# ในโฟลเดอร์โปรเจกต์
claude
```

ข้อความประมาณนี้จะปรากฏ:
```
✅ Claude Code initialized in: C:\Users\TKTF\Desktop\TradingAI
🚀 Ready to start trading analysis
```

#### **ขั้นตอนที่ 4: ตรวจสอบการติดตั้ง MCP**
MCP จะโหลดอัตโนมัติ ลองขอค้นหาข้อมูลหุ้น AAPL:
```
เทรด AAPL วิเคราะห์ข้อมูล
```

ถ้าใช้ได้ คุณจะเห็น tools ต่างๆ เช่น:
- get_stock_data
- analyze_technical
- get_market_news
- generate_trading_signal

---

### 📊 ใช้งาน (Usage)

ขอให้ Claude วิเคราะห์ข้อมูลหุ้น เช่น:

#### **วิเคราะห์หุ้น**
```
วิเคราะห์เทคนิค AAPL ดูตัวชี้วัด RSI, MACD
```

#### **ดึงข่าวตลาด**
```
หาข่าวเกี่ยวกับ Apple ข่าว 5 อันล่าสุด
```

#### **สร้างสัญญาณเทรด**
```
สร้างสัญญาณเทรด AAPL ใช้ strategy momentum
```

---

### 🔧 Configuration Files

**settings.json** - MCP server configuration
**package.json** - Dependencies and scripts
**claude.md** - Project documentation
**mcp/tradingview-server.js** - TradingView MCP server

---

### 🎯 ฟีเจอร์ (Features)

✅ Real-time stock data from TradingView
✅ Technical analysis (RSI, MACD, Bollinger Bands)
✅ Market news aggregation (Yahoo Finance, Reddit)
✅ AI-powered trading signals
✅ Automated price alerts
✅ Multiple trading strategies

---

### 📌 ข้อมูลเพิ่มเติม (Additional Info)

- TradingView MCP ไม่ต้องใช้ API Key
- ข้อมูลดึงมาจาก Yahoo Finance, Reddit, Market News
- รองรับการเทรด 24/7 ด้วย Cron jobs
- สามารถเชื่อมต่อ Telegram สำหรับแจ้งเตือนได้

---

### ⚠️ Troubleshooting

**Q: `/mcp` ไม่ทำงาน?**
A: ตรวจสอบว่า npm install เสร็จแล้ว และ settings.json ถูกต้อง

**Q: ต้องใช้ API Key ของ TradingView?**
A: ไม่ต้อง TradingView MCP ใช้งานฟรี

**Q: ต้องการแจ้งเตือนใน Telegram?**
A: สร้าง Bot ผ่าน BotFather แล้วเพิ่ม Token ใน settings.json

---

### 🎓 สอนเพิ่มเติม

ดูวิดีโอสอนเพิ่มเติม: https://www.youtube.com/watch?v=TilUhQ1t6Ak

---

**Version:** 1.0.0
**Last Updated:** 2026-05-15
