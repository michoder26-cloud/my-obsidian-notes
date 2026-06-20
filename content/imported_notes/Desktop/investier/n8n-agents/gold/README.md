# Gold Trading Multi-Agent (XAU/USD)

ระบบเทรดทองด้วย AI หลายตัว — โครงเหมือน stock workflow แต่เฉพาะทาง

## โครงสร้าง

```
Schedule (19:00 BKK = NY market open)
  ↓
Read Lessons (กรองเฉพาะ "gold" lessons)
  ↓
Fetch Gold + Macro Data
  ├── XAU/USD price + candles (Yahoo: GC=F)
  ├── DXY (Dollar Index) — inverse correlation
  ├── 10Y Bond Yield (^TNX) — real rates
  ├── Forex Factory news (USD high/medium impact)
  └── Fear & Greed Index
  ↓
Calculate Indicators
  ├── Multi-timeframe (D1 + H4)
  ├── RSI 14, MACD, EMA 50/200
  ├── Bollinger Bands, ATR
  └── Fibonacci levels (30-day high/low)
  ↓
🐂 Gold Bull → หาเหตุผลซื้อ + bull target
🐻 Gold Bear → หาเหตุผลขาย + bear target
🤓 Gold Quant → math + risk calculation + entry/SL/TP
  ↓
⚖️ Gold Judge → final signal + trade plan
  ↓
Format → Discord (5 embeds!) + Save to Sheets
```

## ความแตกต่างจาก Stock Workflow

| Stock | Gold |
|-------|------|
| 8 หุ้น/ETF | XAU/USD ตัวเดียว แต่ลึก |
| Finnhub data | Yahoo Finance (GC=F) |
| Fundamental focus (P/E, ROE) | Macro focus (DXY, yields) |
| Daily timeframe | Multi-timeframe (D1 + H4) |
| 1 hour delay OK | Forex 24/5 — เน้นข่าวสด |
| Earnings calendar | Forex Factory news calendar |

## ข้อมูลที่ดึง

### ราคา (Yahoo Finance — ฟรี ไม่ต้อง API key)
- `GC=F` — Gold Futures
- `DX-Y.NYB` — Dollar Index
- `^TNX` — 10-Year Treasury Yield

### ข่าว (Forex Factory — ฟรี ไม่ต้อง API key)
- `https://nfs.faireconomy.media/ff_calendar_thisweek.json`
- กรองเฉพาะ USD + High/Medium impact

### Macro (CNN — ฟรี)
- Fear & Greed Index

## Discord Output (5 embeds)

1. **🥇 Header** — ราคา + macro context
2. **🟢/🔴/🟡 Final Signal** — Entry/SL/TP/R:R
3. **🥊 The Debate** — Bull/Bear/Quant/Hidden Alpha
4. **📊 Technical Snapshot** — D1 + H4 indicators
5. **📰 Upcoming News** — Forex Factory + alerts

## Setup

### 1. Import workflow
- n8n → Import from File → `workflow-gold.json`

### 2. Credentials (ใช้ของเดิมได้)
- Google Sheets (Service Account)
- Discord Webhook URL
- Anthropic API key

### 3. Models แนะนำ
- 🐂 Bull → Sonnet 4.6
- 🐻 Bear → Sonnet 4.6
- 🤓 Quant → **Opus 4.7** (เก่งคณิตสุด)
- ⚖️ Judge → **Opus 4.7**

### 4. Schedule
- Default: **19:00 BKK** (= NY pre-market 8:00 ET)
- ปรับได้ที่ Schedule node

## Key Indicators ที่ใช้

| Indicator | คำนวณยังไง |
|-----------|------------|
| RSI 14 | สูตร RSI มาตรฐาน |
| MACD | EMA12 - EMA26 |
| EMA 50 / 200 | Exponential MA |
| Bollinger Bands | MA20 ± 2σ |
| ATR 14 | Average True Range — ใช้ตั้ง SL |
| Fibonacci | จาก 30-day high/low |

## Risk Management

Quant agent คำนวณให้:
- **Entry** จาก confluence
- **Stop Loss** = ATR × 1.5
- **Take Profit** = ATR × 3 (R:R 1:2)
- **Position size** ตัวอย่าง $10k account, risk 2%

## Memory System

ใช้ Sheets ตัวเดียวกับ stock workflow:
- **Predictions** tab → บันทึกคำแนะนำ
- **Lessons** tab → AI กรองเฉพาะ "gold" lessons เพื่อใช้

หลังใช้ 1-2 เดือน — Reflection workflow จะเขียน lessons เกี่ยวกับ gold โดยอัตโนมัติ

## ข้อควรระวัง

- **Gold trade 24/5** — ตลาดเปิดยาว
- **News volatility** — FOMC, NFP, CPI = spike แรง
- **Spread กว้างตอน Asian session** — เทรดหลัง London open ดีกว่า
- **Yahoo Finance** บางครั้งดีเลย์ 15 นาที
- **Forex Factory feed** อัพเดททุก ~1 ชั่วโมง

## Trading Sessions (BKK time)

| Session | เวลา | Volatility |
|---------|------|-----------|
| Asian | 06:00-15:00 | ต่ำ |
| London | 14:00-23:00 | สูง |
| NY | 19:00-04:00 | สูงสุด |
| **London-NY Overlap** | **19:00-23:00** | **🔥 best** |

→ Schedule 19:00 = ก่อน NY open = วิเคราะห์ทันก่อนตลาดเร่ง
