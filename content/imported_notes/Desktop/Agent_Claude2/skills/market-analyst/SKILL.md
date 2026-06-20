---
name: market-analyst
description: วิเคราะห์ราคาหุ้น/คริปโต/ฟอเร็กซ์แบบครบทุกมิติ — ดึงราคาเรียลไทม์ วิเคราะห์ข่าว วิเคราะห์เทคนิคัล (RSI, MACD, Bollinger, Fibonacci) วิเคราะห์เชิงสถิติ (Monte Carlo, Sharpe Ratio, Volatility) และให้สัญญาณซื้อ/ขาย/ถือพร้อมระดับความเชื่อมั่น ใช้เมื่อผู้ใช้ขอวิเคราะห์ตลาด หุ้น เหรียญคริปโต หรือสินทรัพย์การลงทุน
---

# Market Analyst Skill — วิเคราะห์ตลาดแบบมืออาชีพ

## ⚠️ Disclaimer (ต้องแสดงทุกครั้งก่อนวิเคราะห์)
> การวิเคราะห์นี้เป็นข้อมูลประกอบการตัดสินใจ ไม่ใช่คำแนะนำการลงทุน ตลาดมีความเสี่ยง — ผู้ลงทุนควรศึกษาเพิ่มเติมและรับผิดชอบการตัดสินใจของตนเอง

## ขั้นตอนการวิเคราะห์ (ทำตามลำดับ)

### STEP 1: รับ Input และยืนยันเป้าหมาย
ถามผู้ใช้ (ถ้ายังไม่บอก):
- **Symbol**: ชื่อหุ้น/เหรียญ (เช่น AAPL, BTC, PTT.BK, ETH)
- **Timeframe**: ระยะสั้น (1-7 วัน) / กลาง (1-3 เดือน) / ยาว (6 เดือนขึ้นไป)
- **เป้าหมาย**: เก็งกำไร / ลงทุนระยะยาว / ป้องกันความเสี่ยง

### STEP 2: ดึงข้อมูลเรียลไทม์ (ใช้ WebSearch + WebFetch)

**2.1 ราคาปัจจุบัน + ข้อมูลพื้นฐาน**
ค้นหาด้วย WebSearch:
```
"<symbol> stock price today" หรือ "<symbol> current price USD"
```
แหล่งข้อมูล: Yahoo Finance, Google Finance, CoinGecko, CoinMarketCap, TradingView, 

ต้องดึง:
- ราคาปัจจุบัน, ราคาเปิด, สูงสุด/ต่ำสุดวันนี้
- Volume 24 ชม.
- Market Cap
- การเปลี่ยนแปลง 24h, 7d, 30d, 1y
- 52-week high/low

**2.2 ข่าวล่าสุด (วิเคราะห์ Sentiment)**
WebSearch:
```
"<symbol> news today"
"<symbol> latest news this week"
"<company-name> earnings/announcement"
```
แยกข่าวเป็น 3 กลุ่ม:
- 🟢 **Bullish (ข่าวดี)**: earnings beat, partnerships, product launches, upgrades
- 🔴 **Bearish (ข่าวร้าย)**: lawsuits, downgrades, scandals, regulatory issues
- 🟡 **Neutral**: ข่าวทั่วไป

**2.3 ข้อมูล Macro (สำหรับ context)**
- Fed rate, CPI, GDP (สำหรับหุ้น US)
- Bitcoin dominance, Fear & Greed Index (สำหรับ crypto)


### STEP 3: Technical Analysis (วิเคราะห์เทคนิคัล)

คำนวณและตีความ indicators ต่อไปนี้ — **อธิบายสูตรและความหมายเสมอ**:

**3.1 Trend Indicators**
- **Moving Averages**: SMA 20, 50, 200 — ราคาอยู่เหนือ/ใต้ MA?
  - Golden Cross (50 ตัด 200 ขึ้น) = สัญญาณซื้อแรง
  - Death Cross (50 ตัด 200 ลง) = สัญญาณขายแรง
- **EMA 12, 26**: ตอบสนองเร็วกว่า SMA

**3.2 Momentum Indicators**
- **RSI (14)**: 
  - > 70 = Overbought (อาจกำลังจะปรับฐาน)
  - < 30 = Oversold (อาจจะกำลังจะเด้ง)
  - Divergence = สัญญาณกลับตัว
- **MACD**: 
  - MACD line ตัด Signal line ขึ้น = bullish
  - Histogram เปลี่ยนทิศ = momentum shift
- **Stochastic (14,3,3)**: confirm RSI

**3.3 Volatility Indicators**
- **Bollinger Bands (20, 2)**:
  - แตะ Upper Band = อาจ overbought
  - แตะ Lower Band = อาจ oversold
  - Squeeze = กำลังจะมี breakout
- **ATR (14)**: วัดความผันผวน → ใช้ตั้ง Stop Loss

**3.4 Volume Analysis**
- **OBV (On-Balance Volume)**: ยืนยันแนวโน้มราคา
- **Volume Spike**: > 2x ค่าเฉลี่ย = มีการเข้า/ออกมาก

**3.5 Support & Resistance**
- หา **Fibonacci Retracement** (23.6%, 38.2%, 50%, 61.8%, 78.6%)
- หา **Pivot Points** (Daily, Weekly)
- หาแนวรับ-แนวต้านจาก swing high/low

### STEP 4: Quantitative Analysis (เชิงคณิตศาสตร์)

**4.1 Statistical Metrics**
- **Volatility (σ)**: คำนวณจาก daily returns
  - σ = √(Σ(r - μ)² / n)
- **Sharpe Ratio**: (Return - RiskFreeRate) / σ
  - > 1 = ดี, > 2 = ดีมาก, > 3 = ยอดเยี่ยม
- **Beta**: ความสัมพันธ์กับตลาด (S&P500 หรือ SET)
  - β > 1 = ผันผวนกว่าตลาด

**4.2 Monte Carlo Simulation (ทำในใจ/เชิงคุณภาพ)**
ประมาณ 1,000 scenarios:
- Best case (95th percentile): ?
- Expected (50th percentile): ?
- Worst case (5th percentile): ?

**4.3 Risk Metrics**
- **Value at Risk (VaR 95%)**: ขาดทุนสูงสุดที่อาจเกิดใน 95% ของกรณี
- **Maximum Drawdown**: การลดลงสูงสุดจากจุดสูงสุด
- **Risk/Reward Ratio**: (Target - Entry) / (Entry - StopLoss)
  - ต้อง ≥ 1:2 ถึงคุ้มเทรด

**4.4 Correlation Analysis**
- ความสัมพันธ์กับสินทรัพย์อื่น (BTC, SPY, Gold, DXY)
- ใช้กระจายความเสี่ยง

### STEP 5: Advanced Techniques (เทคนิคขั้นสูง)

**5.1 Multi-Timeframe Analysis (MTF)**
ตรวจสอบ trend ใน 3 timeframes:
- Daily (ภาพใหญ่)
- 4H (จุดเข้า)
- 1H (timing แม่นยำ)
→ ทั้ง 3 ต้องสอดคล้องกัน = สัญญาณแข็งแกร่ง

**5.2 Order Flow & Market Structure**
- Higher Highs + Higher Lows = Uptrend
- Lower Highs + Lower Lows = Downtrend
- Break of Structure (BOS) = trend change confirmation
- Liquidity zones (จุดที่ stop loss กระจุกตัว)

**5.3 Sentiment Quantification**
ให้คะแนน sentiment 0-100:
- News sentiment (40% weight)
- Social media buzz (20%)
- Fear & Greed Index (20%)
- Options flow / funding rate (20%)

**5.4 Regime Detection**
ตลาดอยู่ในโหมดไหน?
- Trending (ใช้ MA, MACD)
- Ranging (ใช้ RSI, Bollinger)
- High Volatility (ลด position size)
- Low Volatility (รอ breakout)

### STEP 6: สังเคราะห์ผล (Synthesis)

ให้คะแนนแต่ละด้าน 1-10:
| ด้าน | คะแนน | น้ำหนัก |
|------|-------|---------|
| Technical | ?/10 | 30% |
| Fundamental/News | ?/10 | 25% |
| Sentiment | ?/10 | 15% |
| Macro Context | ?/10 | 15% |
| Risk/Reward | ?/10 | 15% |

**Final Score = Σ(score × weight)**
- 8-10 = STRONG BUY 🟢
- 6-7.9 = BUY 🟢
- 4-5.9 = HOLD 🟡
- 2-3.9 = SELL 🔴
- 0-1.9 = STRONG SELL 🔴

### STEP 7: Output Template

แสดงผลตามรูปแบบนี้:

```markdown
# 📊 Market Analysis: <SYMBOL>
*วิเคราะห์ ณ วันที่ YYYY-MM-DD HH:MM*

## 💰 ราคาปัจจุบัน
- **Price**: $XXX.XX
- **24h Change**: +X.XX% / -X.XX%
- **Volume**: $XXX M
- **Market Cap**: $XXX B

## 📰 ข่าวสำคัญ (24-48 ชม.)
🟢 **ข่าวดี**:
- ...

🔴 **ข่าวร้าย**:
- ...

**Sentiment Score**: XX/100

## 📈 Technical Analysis
- **Trend** (Daily): Up/Down/Sideways
- **RSI(14)**: XX.X (สถานะ)
- **MACD**: Bullish/Bearish crossover
- **MA**: ราคาอยู่เหนือ/ใต้ MA200
- **Support**: $XXX, $XXX
- **Resistance**: $XXX, $XXX

## 🔢 Quantitative Metrics
- **Volatility (30d)**: XX%
- **Sharpe Ratio**: X.XX
- **Beta**: X.XX
- **VaR (95%)**: -X.XX%
- **Max Drawdown (1y)**: -XX%

## 🎯 Trade Setup (ถ้าผู้ใช้ขอ)
- **Entry**: $XXX - $XXX
- **Stop Loss**: $XXX (-X.X%)
- **Take Profit 1**: $XXX (+X.X%)
- **Take Profit 2**: $XXX (+X.X%)
- **Risk/Reward**: 1:X.X
- **Position Size**: แนะนำ X% ของพอร์ต

## 🏆 Final Verdict
**Score: X.X/10**
**Signal**: 🟢 BUY / 🟡 HOLD / 🔴 SELL
**Confidence**: HIGH / MEDIUM / LOW

**เหตุผลสรุป**:
1. ...
2. ...
3. ...

## ⚠️ ความเสี่ยงที่ต้องระวัง
- ...
- ...

---
*ข้อมูลนี้ไม่ใช่คำแนะนำการลงทุน*
```

## กฎสำคัญ
1. **ใช้ WebSearch/WebFetch ดึงข้อมูลจริงเสมอ** — ห้ามมั่วราคา
2. **อ้างอิงแหล่งข้อมูล** ทุกครั้งที่อ้างถึงตัวเลข
3. **บอกข้อจำกัด** — Claude ไม่มี real-time data feed, ข้อมูลอาจช้า 15-30 นาที
4. **ไม่รับประกันผล** — ใช้คำว่า "อาจจะ", "มีแนวโน้ม", "สัญญาณบ่งชี้"
5. **อธิบายเป็นภาษาไทย** เป็นหลัก แต่คงศัพท์เทคนิคอังกฤษ
6. **ถามก่อนถ้าข้อมูลไม่พอ** — เช่น timeframe, ขนาดพอร์ต
7. **เตือนความเสี่ยงเสมอ** — ไม่มีกลยุทธ์ใดชนะ 100%
