# 🤖 AI Trading with Claude Code & TradingView MCP

เทรดด้วย AI แบบฟรี! ระบบการวิเคราะห์ตลาดอัตโนมัติด้วย Claude Code และ TradingView MCP

## 🎯 ตัวอย่างการใช้งาน

### ขั้นตอน 1: เปิด Claude Code
```bash
cd C:\Users\TKTF\Desktop\TradingAI
claude
```

### ขั้นตอน 2: ใช้คำสั่ง MCP
```
/mcp
```

### ขั้นตอน 3: วิเคราะห์หุ้น
```
/analyze AAPL
```

---

## 📊 เครื่องมือที่มีให้ (Tools)

| ชื่อเครื่องมือ | คำอธิบาย |
|---|---|
| `get_stock_data` | ดึงข้อมูลหุ้นแบบ Real-time |
| `analyze_technical` | วิเคราะห์ RSI, MACD, Bollinger |
| `get_market_news` | ข่าวตลาดจาก Yahoo, Reddit |
| `generate_trading_signal` | สร้างสัญญาณซื้อ/ขาย |

---

## 🚀 เริ่มต้นใช้งาน

```bash
# 1. ติดตั้ง dependencies
npm install

# 2. เปิด Claude Code
claude

# 3. ตรวจสอบ MCP
/mcp

# 4. วิเคราะห์หุ้นแรก
/analyze TSLA
```

---

## ⚙️ Configuration

แก้ไขไฟล์ `settings.json` เพื่อปรับแต่ง:

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["./mcp/tradingview-server.js"]
    }
  }
}
```

---

## 📚 ไฟล์ที่สำคัญ

```
TradingAI/
├── claude.md                 # Project docs
├── package.json              # Dependencies
├── settings.json             # MCP config
├── trading-analysis.js       # Main script
├── SETUP_GUIDE.md           # Setup instructions
├── mcp/
│   └── tradingview-server.js # MCP server
└── README.md               # This file
```

---

## 🎓 สอนเพิ่มเติม

👉 **ดูวิดีโอเสอนเต็ม:** https://www.youtube.com/watch?v=TilUhQ1t6Ak

---

## ❓ คำถามบ่อย

**Q: ต้องจ่ายเงินหรือไม่?**
A: ไม่ต้อง ใช้งานฟรี 100%

**Q: ต้อง API Key ของ TradingView?**
A: ไม่ต้อง

**Q: สามารถเทรด 24/7 ได้ไหม?**
A: ได้ ด้วยการตั้ง Cron jobs

**Q: เชื่อมต่อ Telegram ได้ไหม?**
A: ได้ ต้องเพิ่ม Bot Token ใน settings.json

---

## 📞 Support

หากมีปัญหา ลองเช็ค:
1. Node.js version >= 18
2. npm dependencies ติดตั้งถูกต้อง
3. settings.json มีรูปแบบถูกต้อง
4. Path ของ mcp/tradingview-server.js ถูกต้อง

---

**Developed with ❤️ using Claude Code**
**Version 1.0.0 | 2026-05-15**
