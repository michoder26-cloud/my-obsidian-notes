# XAU/USD Technical Analysis

Agent ที่ดึงข้อมูลทองคำ (XAU/USD) จาก Alpha Vantage API วิเคราะห์ด้วย Technical Indicators แล้วสร้างกราฟ + ให้สัญญาณซื้อ/ขาย

## 📋 Requirements

- Python 3.8+
- API Key จาก Alpha Vantage (ฟรี)

## 🚀 Quick Start

### 1. ดึง API Key

ไปที่: https://www.alphavantage.co/support/#api-key
- กรอก email
- ได้ API key ส่งเข้า email ทันที

### 2. Setup Project

```bash
cd Desktop\xau-analyzer

# สร้าง virtual environment
python -m venv venv
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 3. เก็บ API Key

แก้ไข `.env` file:
```
ALPHA_VANTAGE_API_KEY=your_actual_api_key_here
```

### 4. Run Script

```bash
python xau_analyzer.py
```

Output:
- 📊 กราฟ PNG (XAU_USD_Analysis_YYYYMMDD_HHMMSS.png)
- 📈 Signal: 🟢 BUY / 🔴 SELL / 🟡 HOLD
- 📊 Technical indicators: RSI, MACD, Bollinger Bands, EMA

## 📊 Technical Indicators

| Indicator | What it does |
|-----------|-------------|
| **RSI (14)** | แสดง Overbought/Oversold (30-70) |
| **MACD** | Trend momentum |
| **Bollinger Bands** | Support/Resistance levels |
| **EMA 12/26** | Moving averages trend |

## 🎯 Signal Logic

### 🟢 BUY
- RSI < 30 (oversold)
- MACD crosses above signal line
- EMA 12 > EMA 26

### 🔴 SELL
- RSI > 70 (overbought)
- MACD crosses below signal line
- EMA 12 < EMA 26

### 🟡 HOLD
- ไม่มีสัญญาณชัดเจน

## ⚠️ Note

- Free tier: 5 requests/min, 500 requests/day
- Data lag: 15-20 minutes (free tier)
- Confidence: 0-100% based on indicator strength

## 📝 Example Output

```
==================================================
XAU/USD Technical Analysis
==================================================
Current Price: $2,450.50
Signal: 🟢 BUY
Confidence: 85.5%
RSI (14): 28.3
MACD: 0.000145
EMA 12/26 Ratio: 1.25%
==================================================
📈 Chart saved: XAU_USD_Analysis_20240115_143022.png
✅ Done!
```

## 🔗 Resources

- Alpha Vantage: https://www.alphavantage.co/
- Technical Analysis Library: https://github.com/bukosabino/ta
- Matplotlib Docs: https://matplotlib.org/
