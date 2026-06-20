# 🚀 Stock Analysis System - Quick Start

**ง่ายๆ เลย**

---

## ✅ ขั้น 1: Setup (ทำแค่ครั้งเดียว)

```powershell
# 1. ไปที่ TradingAI folder
cd C:\Users\TKTF\Desktop\TradingAI

# 2. Install dependencies
pip install -r requirements.txt

# 3. Edit .env ใส่ API key
# OPENROUTER_API_KEY=sk-or-...
```

---

## ⚡ ขั้น 2: รัน

```powershell
python stock_analysis_system.py
```

**แค่นี้!** ✅

---

## 📊 ได้ผลลัพธ์:

```
📊 STOCK ANALYSIS: GOOGL
================================================

[1/4] 🔍 Fetching data...
[2/4] 📈 Analyzing fundamentals...
[3/4] 💭 Analyzing sentiment...
[4/4] 🎯 Generating recommendation...

🎯 Recommendation: BUY
   Confidence: 85%
   Price Target: $220
   Upside: 12%

... (เต็มไป)
```

---

## 🎯 เปลี่ยนหุ้น:

Edit ที่ท้ายของ `stock_analysis_system.py`:

```python
stocks = ["GOOGL", "AAPL", "MSFT"]  # ← เปลี่ยนตรงนี้

for symbol in stocks[:1]:  # [:1] = 1 หุ้น, [:3] = 3 หุ้น
    orchestrator.analyze_stock(symbol)
```

---

## ❓ ติดขัด?

### API Key ไม่ตั้ง:
```
1. ไป https://openrouter.ai
2. Sign up (ฟรี)
3. Copy key
4. ใส่ใน .env
```

### Timeout/Error:
```powershell
# ลองรัน test ก่อน
python test_system.py
```

---

## 📈 ใช้งาน:

```powershell
# วิเคราะห์ Apple
python stock_analysis_system.py
# (แก้ code เป็น AAPL)

# วิเคราะห์ Microsoft
python stock_analysis_system.py
# (แก้ code เป็น MSFT)
```

---

**ไม่ซับซ้อน ทำแบบนี้เอาหรือ?** 👍
