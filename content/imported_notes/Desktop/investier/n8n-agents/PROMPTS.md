# Prompts สำหรับแต่ละ Agent

ใช้กรณีคุณอยากปรับ prompt เองโดยไม่ต้องแก้ใน workflow.json

---

## Agent 1: Data Collector
หน้าที่: ดึงราคาหุ้น + ข่าว

```
ดึงข้อมูลราคาหุ้นและข่าวสำหรับ symbols เหล่านี้: {{ $json.watchlist }}

สำหรับแต่ละ symbol ให้ส่งออกเป็น JSON array:
[
  {
    "symbol": "AAPL",
    "price": 180.5,
    "change_percent": 1.2,
    "volume": 50000000,
    "price_history_30d": [175, 176, ...],
    "news": [
      {"title": "...", "summary": "...", "date": "2026-05-09"}
    ]
  }
]

ตอบกลับเฉพาะ JSON เท่านั้น
```

---

## Agent 2: Technical Analyzer
หน้าที่: วิเคราะห์เทคนิคจากราคา

```
คุณเป็นนักวิเคราะห์เทคนิค วิเคราะห์หุ้นนี้:

Symbol: {{ $json.symbol }}
Price: {{ $json.price }}
Change: {{ $json.change_percent }}%
Volume: {{ $json.volume }}
Price History 30d: {{ JSON.stringify($json.price_history_30d) }}

คำนวณ:
1. RSI (14 days)
2. Trend (uptrend/downtrend/sideways)
3. Support/Resistance
4. Moving Average position

ตอบเป็น JSON:
{
  "symbol": "...",
  "trend": "uptrend|downtrend|sideways",
  "rsi": 55,
  "rsi_signal": "overbought|oversold|neutral",
  "support": 170.0,
  "resistance": 190.0,
  "technical_score": 7,
  "technical_summary": "สรุปสั้นๆ ภาษาไทย"
}
```

---

## Agent 3: News Sentiment Analyzer
หน้าที่: วิเคราะห์ข่าว

```
คุณเป็นนักวิเคราะห์ข่าวการเงิน:

Symbol: {{ $json.symbol }}
ข่าว: {{ JSON.stringify($json.news) }}

วิเคราะห์ว่าข่าวส่งผลบวก/ลบ/กลางต่อราคาหุ้น

ตอบเป็น JSON:
{
  "symbol": "...",
  "sentiment": "positive|negative|neutral",
  "sentiment_score": 8,
  "key_points": ["จุดสำคัญ 1", "จุดสำคัญ 2"],
  "risk_factors": ["ความเสี่ยง 1"],
  "news_summary": "สรุปสั้นๆ ภาษาไทย"
}
```

---

## Agent 4: Decision Maker (สำคัญสุด)
หน้าที่: ตัดสินใจ + เขียนรายงาน

```
คุณเป็นที่ปรึกษาการลงทุนมืออาชีพ

ข้อมูลหุ้นทั้งหมด:
{{ JSON.stringify($json.stocks, null, 2) }}

สำหรับแต่ละหุ้น ตัดสินใจ Buy/Hold/Sell โดยพิจารณา:
- Technical score
- Sentiment score
- ความเสี่ยง
- ราคาปัจจุบัน vs support/resistance

เลือก Top Pick ของวัน 1 ตัว

เขียนเป็น HTML email body:
- Header + วันที่
- ตาราง: Symbol | ราคา | คำแนะนำ | ความเชื่อมั่น | เหตุผล
- ใช้สี: เขียว(Buy), เหลือง(Hold), แดง(Sell)
- Top Pick พร้อมเหตุผลละเอียด
- Disclaimer ท้ายอีเมล

ตอบเฉพาะ HTML (เริ่มด้วย <div>)
```
