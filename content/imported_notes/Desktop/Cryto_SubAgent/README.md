# 🏆 BTC/USDT MMTC v2.0: The Holy Grail Strategy

A sophisticated **multi-agent AI trading system** for Crypto (BTC/USDT). Codenamed **MMTC v2.0 (Market Maker Trap Catcher)**, this system combines Institutional Smart Money Concepts (SMC), dynamic Fibonacci Circles, and ruthless Risk Management to hunt for high-probability sniper entries.

---

## 🔥 The Holy Grail Strategy (SMC + Fibo Circles)
The core of MMTC v2.0 is built on identifying where retail traders lose and where Market Makers (Institutions) enter.

### 📐 Structural Mapping
1. **Dynamic CHoCH / BOS Detection:** The bot uses a 5-bar fractal algorithm to identify valid Swing Highs and Swing Lows, dynamically mapping Market Structure.
2. **Left-to-Right Wick Anchoring:** Fibonacci Retracements and Circles are drawn precisely from the origin swing to the completion swing, capturing 100% of the liquidity wicks.

### 🎯 The 3 Institutional Zones (Price Tiers)
*   **Tier 1: Equilibrium Zone (`23.6% - 38.2%`)** - Trend continuation. Triggers if MACD quickly confirms the pullback.
*   **Tier 2: Fair Value Pool (`50.0% - 61.8%`)** - The primary Day Trading zone. Discount for buyers, Premium for sellers.
*   **Tier 3: Market Maker All-In Zone (`78.6% - 88.7%`)** - The ultimate liquidity sweep zone. This is the final defense level before a CHoCH invalidates the trend.

### 💥 God Mode: Lot x3 Override
When the price enters the **Tier 3 (78.6% - 88.7%)** zone and momentum (RSI) confirms exhaustion:
1. **Aggressive Sizing:** The bot multiplies its normal lot size by **3x** (Risking 18% instead of 6%).
2. **Tight Stop Loss:** SL is compressed to a razor-thin 100 USDT distance (just beyond the 100% Fibo invalidation line).
3. **1:10 Risk-Reward:** Targeting massive extensions (161.8%), a single winning trade in this zone can grow the portfolio by **180%**.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│          Master Orchestrator                            │
│  (รวมประสานงาน + ตัดสินใจสุดท้าย)                        │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┼───────────┬───────────┐
         ▼           ▼           ▼           ▼
    ┌────────┐  ┌────────┐ ┌────────┐ ┌──────────┐
    │ News   │  │ Bull   │ │ Bear   │ │ CEO      │
    │ Analyst│  │ Agent  │ │ Agent  │ │ Agent    │
    └────────┘  └────────┘ └────────┘ └──────────┘
         │           │           │           │
         └───────────┼───────────┴───────────┘
                     │
              ┌──────▼──────┐
              │ Backtester  │
              │(BinanceLive)│
              └─────────────┘
```

## 📋 Agents & Responsibilities

### 1. 📰 **News Analyst**
- Parses Crypto macroeconomic news and event data (Fed speeches, CPI, regulation changes).
- Enforces "News Blackouts" to prevent trading during extreme volatility.

### 2. 🐂 **Bull Agent (The Prosecutor for Longs)**
- Searches exclusively for bullish SMC setups (FVG, Order Blocks, Liquidity Sweeps).
- Activates Setup E (Fibo Circle Support) to call for Lot x3 Longs.

### 3. 🐻 **Bear Agent (The Prosecutor for Shorts)**
- Searches exclusively for bearish setups.
- Activates Setup E (Fibo Circle Resistance) to call for Lot x3 Shorts.

### 4. 👔 **CEO Agent (The Ultimate Decision Maker)**
- Listens to both Bull and Bear cases.
- Requires a strict conviction score (>= 78%) to approve a trade.
- Reads `[OVERRIDE_LOT_MULTIPLIER=3.0]` flags to authorize aggressive execution.

---

## 🛡️ Advanced Risk & Trade Management

*   **Sniper Engine (Golden Hours):** Trades specifically during highest volatility overlap sessions (US Session Open & Asia/London overlap) to avoid quiet sideways market fakeouts.
*   **3-Phase Trailing Stop:** 
    1. **Breakeven:** Once profit reaches 300 USDT, SL moves to entry.
    2. **Lock 1:** At 450 USDT profit, locks in 180 USDT.
    3. **Lock 2:** At 750 USDT profit, locks in 360 USDT.
*   **ATR Dynamic SL:** During normal trades (Tier 1/2), SL distance expands/contracts based on current market volatility.

---

## 📁 Project Layout

ดูรายละเอียดโฟลเดอร์ใน [`PROJECT_STRUCTURE.md`](PROJECT_STRUCTURE.md)

```
Cryto_SubAgent/
├── main.py, auto_trader.py    # จุดเข้าใช้งาน
├── src/                       # ระบบ multi-agent
├── strategies/                # MMTC v4 + backtest หลายปี
├── scripts/                   # ทดลอง / optimize / analyze
├── results/                   # ผล backtest (gitignore)
├── docs/                      # คู่มือ + เอกสารกลยุทธ์
├── stock/                     # ระบบหุ้น (แยก)
└── archive/                   # เวอร์ชันเก่า + scratch
```

---

## 🚀 Quick Start

### 1. Prerequisites (สเปกและความต้องการของระบบ)
- **ระบบปฏิบัติการ (OS)**: บอทนี้ใช้งาน API ของ **Binance / Bybit** ซึ่งรองรับระบบปฏิบัติการที่หลากหลายทั้ง **Windows, macOS และ Linux**
- **บัญชีเทรด Crypto**: ต้องเปิดบัญชีกับ Exchange (เดโม/Testnet หรือ พอร์ตจริง) และเตรียม API Key/Secret Key
- **การตั้งค่า API**: 
  1. เปิดใช้งานการเข้าถึง API ในการตั้งค่าบัญชีของคุณ
  2. เปิดใช้งานสิทธิ์ **"Enable Futures"** หรือ **"Enable Trading"**
  3. เพิ่มสัญลักษณ์คู่เทรดคริปโต (เช่น `BTCUSDT` หรือ `ETHUSDT`) เข้ามาในคู่เหรียญที่สนใจวิเคราะห์
- **การติดตั้งแพ็กเกจ**:
  ```bash
  pip install -r requirements.txt
  ```

### 2. Setup Environment (ตั้งค่าไฟล์ระบบ)
ก๊อบปี้ไฟล์ตัวอย่างการตั้งค่าไปสร้างเป็นไฟล์ `.env`:
```bash
cp .env.example .env
```
เปิดไฟล์ `.env` ขึ้นมาเพื่อกรอกข้อมูลที่จำเป็น:
*   `OPENROUTER_API_KEY`: คีย์สำหรับเชื่อมต่อ AI ของ OpenRouter (ใช้ขับเคลื่อนการวิเคราะห์ตัดสินใจของ CEO, Bull, Bear และประเมินบทเรียนเทรด)
*   `DISCORD_WEBHOOK_URL`: ลิงก์ Discord Webhook สำหรับรับการรายงานแบบเรียลไทม์ (แจ้งเตือนเปิดออเดอร์, ปิดออเดอร์, การลาก Trailing Stop, รายงานรายสัปดาห์)
*   **การตั้งค่า Crypto API (Live / Paper Trading เท่านั้น)**:
     *   `BINANCE_API_KEY`: API Key ของ Binance
     *   `BINANCE_API_SECRET`: API Secret ของ Binance
     *   `BYBIT_API_KEY`: API Key ของ Bybit (หากเลือกใช้ Bybit)
     *   `BYBIT_API_SECRET`: API Secret ของ Bybit (หากเลือกใช้ Bybit)
     *   `CRYPTO_SYMBOL`: สัญลักษณ์เหรียญคริปโต (เช่น `BTCUSDT`)
*   **การตั้งค่า Trailing Stop**:
    *   `USE_TRAILING_STOP`: ตั้งเป็น `true` เพื่อเปิดใช้ Trailing Stop แบบ 3 ระยะ
    *   `TRAIL_BREAKEVEN_TRIGGER`: สัดส่วนการขยับ SL ไปหน้าทุน (ค่าเริ่มต้น: `1.0` เท่าของระยะ SL)
    *   `TRAIL_LOCK_TRIGGER`: สัดส่วนการขยับ SL ไปล็อกกำไร 50% (ค่าเริ่มต้น: `2.0` เท่าของระยะ SL)

### 3. Running Commands (คำสั่งการใช้งาน)

#### 🧪 3.1 การรัน Backtest (ทดสอบย้อนหลัง)
คุณสามารถรันตัวทดสอบย้อนหลังเพื่อวิเคราะห์ประสิทธิภาพของบอทด้วยสถิติต่างๆ:
```bash
# แบบรันการทดสอบเบื้องต้น (ใช้ไฟล์เช็ค Trailing Stop)
python scripts/check_24_25_trailing_stop.py

# MMTC backtest หลายปี
python strategies/run_multiyear_backtest.py

# หรือรันผ่าน main.py เพื่อกำหนดช่วงเวลาและสัญลักษณ์เอง:
python main.py backtest --start-date 2026-01-01 --end-date 2026-05-01 --interval 1h --risk-pct 15.0 --symbol BTCUSDT
```

#### 📄 3.2 การรัน Paper Trading (จำลองการเทรดสดบนบัญชี Demo/Testnet)
รันบอทวิเคราะห์ตลาดสดและจำลองการเทรดโดยใช้ยอดเงินของบัญชี Demo/Testnet:
```bash
python main.py paper --interval 60 --symbol BTCUSDT
```

#### ⚠️ 3.3 การรัน Live Trading (เทรดจริงบนพอร์ตเงินจริง)
รันบอทให้เข้าคุมพอร์ตและเปิดออเดอร์ส่งคำสั่งซื้อขายจริงด้วยเงินจริงบนบัญชีจริง:
```bash
python main.py live --confirm --interval 60 --symbol BTCUSDT
```
> **หมายเหตุความปลอดภัย**: การรันโหมด `live` จำเป็นต้องระบุแฟล็ก `--confirm` เสมอ เพื่อเป็นการยืนยันความเสี่ยงก่อนที่ระบบจะเริ่มทำงาน

---

## ⚠️ Important Notes
**Disclaimer**: This is a highly aggressive, institutional-grade mathematical algorithm utilizing compounding risk. The "Lot x3 Override" carries extreme risk per trade. Use on a Testnet/Demo Account first. Past performance does not guarantee future results.

**Created for elite BTC/USDT trading.**
