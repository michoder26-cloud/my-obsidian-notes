# Investier — Stock Analysis Web App

เว็บวิเคราะห์หุ้น (US/Global) ด้วย Python + React  
ดึงราคาจาก yfinance, ข่าวจาก yfinance + RSS, วิเคราะห์ technical + sentiment, ให้คำแนะนำ ซื้อ/ถือ/ขาย

## Features

- 🔍 **ค้นหาหุ้น** — Search ด้วยชื่อหรือ ticker (AAPL, NVDA, TSLA …)
- 📈 **ราคาปัจจุบัน + กราฟ** — 5D / 1M / 3M / 6M / 1Y / 5Y
- 📰 **ข่าวดี/ข่าวร้าย** — รวมจาก yfinance + Yahoo RSS, วิเคราะห์ sentiment ด้วย VADER
- 📊 **Technical indicators** — RSI, MACD, SMA(20/50/200), Bollinger Bands
- 🎯 **Buy / Hold / Sell recommendation** — รวมคะแนน technical + sentiment
- 🤖 **Claude AI analysis** (optional) — สรุป/วิเคราะห์ข่าวเชิงลึก
- 💼 **Company info** — sector, industry, market cap, P/E, EPS, beta ฯลฯ

## โครงสร้าง

```
investier/
├── backend/         FastAPI + yfinance + VADER + Claude
└── frontend/        React + Vite + Recharts
```

## วิธีรัน

### 1. Backend (FastAPI)

```powershell
cd backend
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt

# (optional) เปิดใช้ Claude
copy .env.example .env
# แก้ .env ใส่ ANTHROPIC_API_KEY=sk-ant-...

python main.py
```

Backend จะรันที่ `http://localhost:8000`  
API docs: `http://localhost:8000/docs`

### 2. Frontend (React)

เปิดอีก terminal:

```powershell
cd frontend
npm install
npm run dev
```

เปิดเว็บที่ `http://localhost:5173`

## API Endpoints

| Method | Path | คำอธิบาย |
|---|---|---|
| GET | `/api/health` | health check + claude availability |
| GET | `/api/search?q=apple` | ค้นหาหุ้นจากชื่อ/ticker |
| GET | `/api/stock/{symbol}` | ราคาปัจจุบัน + ข้อมูลบริษัท |
| GET | `/api/stock/{symbol}/history?period=6mo` | ราคาย้อนหลัง |
| GET | `/api/stock/{symbol}/news` | ข่าว + sentiment |
| GET | `/api/stock/{symbol}/analysis?useAi=true` | วิเคราะห์ครบชุด |

## Recommendation logic

คะแนนรวม -100 ถึง +100:
- Technical (-70 ถึง +70): RSI, MACD, Trend (SMA50/200), Bollinger
- Sentiment (-30 ถึง +30): ค่าเฉลี่ย sentiment score ของข่าวล่าสุด

| คะแนน | คำแนะนำ |
|---|---|
| ≥ +40 | STRONG BUY |
| +15 ถึง +40 | BUY |
| -15 ถึง +15 | HOLD |
| -40 ถึง -15 | SELL |
| ≤ -40 | STRONG SELL |

## ⚠️ Disclaimer

แอปนี้เพื่อการศึกษาเท่านั้น **ไม่ใช่คำแนะนำการลงทุน** การตัดสินใจซื้อขายหุ้นจริงควรปรึกษาผู้เชี่ยวชาญและทำการบ้านเพิ่มเติม
