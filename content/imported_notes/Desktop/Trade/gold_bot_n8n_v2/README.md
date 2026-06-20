# 🥇 Gold Multi-Agent Bot v2 — n8n (AI Agent Pattern)

อัปเกรดจาก v1 — ใช้ **n8n AI Agent node + Anthropic Chat Model** sub-node + เพิ่ม **Quant agent** ตัวที่ 4 + **Google Sheets memory** เพื่อให้ agent เรียนรู้จากเทรดเก่า

## โครงสร้าง Workflow

```
[Schedule]
    ↓
[Read Lessons (Sheets)] → [Format Lessons]
                              ↓
                  [Fetch Stock Data + Indicators]
                              ↓
                    🐂 Agent Bull ─── 🤖 Anthropic Chat Model
                              ↓
                    🐻 Agent Bear ─── 🤖 Anthropic Chat Model
                              ↓
                    📊 Agent Quant ── 🤖 Anthropic Chat Model
                              ↓
                    ⚖️ Agent Judge ── 🤖 Anthropic Chat Model
                                    └── 📋 Output Parser (JSON schema)
                              ↓
                       [Format Output]
                              ↓
                    [Split Rows for Sheets]
                          ↙        ↘
              [Send to Discord]  [Save to Sheets]
```

## ความต่างจาก v1

| | v1 | v2 |
|--|---|---|
| Agent node | HTTP Request ดิบ | **AI Agent node** (สีส้ม 🤖) |
| LLM connection | hardcoded body | **Anthropic Chat Model** sub-node |
| Number of agents | 3 (Bull/Bear/Judge) | **4** (+ Quant) |
| Memory | ไม่มี | **Google Sheets** (lessons จากเทรดเก่า) |
| Output structure | manual JSON | **Structured Output Parser** node |
| Save trades | ไม่มี | **Append to Sheets** ทุกครั้ง |

## Setup

### 1. ติดตั้ง n8n + langchain nodes

n8n v1.0+ มี langchain nodes built-in อยู่แล้ว — ถ้าใช้ self-host เก่ากว่านั้น อัปเดตเป็นล่าสุด:

```powershell
# ถ้าติดตั้งผ่าน npm
npm update n8n -g

# Docker
docker pull n8nio/n8n:latest
```

### 2. Import Workflow

1. n8n UI → **Workflows** → **+ Add workflow** → **⋯** → **Import from File**
2. เลือก [workflow.json](workflow.json)

### 3. สร้าง Anthropic Credential

ใน n8n langchain Anthropic node ใช้ credential ประเภท **"Anthropic API"** (ไม่ใช่ Header Auth):

1. คลิก node **"Anthropic Chat Model (Bull)"**
2. คลิก credential dropdown → **+ Create New Credential**
3. ใส่ **API Key:** `sk-ant-...`
4. **Save** ตั้งชื่อเช่น "Anthropic"
5. ทำซ้ำที่ Bear/Quant/Judge **หรือ** เลือก credential เดิมจาก dropdown

### 4. สร้าง Google Sheet สำหรับ Lessons

1. สร้าง Google Sheet ใหม่ ชื่อ "Gold Trading Journal"
2. สร้างแท็บชื่อ **"Lessons"** (ตรงกับ workflow)
3. แถวแรกเป็นหัวคอลัมน์:

```
date | action | ticker | confidence | entry | sl | tp | rr | bull_score | bear_score | quant_alignment | timeframe | position_oz | position_lots | risk_usd | max_loss | target_profit | reasoning | key_risks | outcome | lesson
```

4. คัดลอก **Sheet ID** จาก URL (เช่น `https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit`)

### 5. ตั้ง Google Sheets Credential

1. คลิก node **"Read Lessons"** → credential
2. **+ Create New Credential** → **Google Sheets OAuth2 API**
3. ทำตาม OAuth flow (ต้องมี Google account)
4. ทำซ้ำที่ "Save to Sheets" หรือ reuse

5. ในทั้ง 2 nodes แทนที่ `REPLACE_WITH_GOOGLE_SHEET_ID` ด้วย Sheet ID จริง

### 6. ตั้ง Environment Variables

n8n Settings → Variables (Cloud) หรือ `.env` (self-host):

```
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
ACCOUNT_USD=10000
RISK_PCT=1.0
```

### 7. ทดสอบ

1. กด **Execute Workflow** — ดู nodes รันไล่ลงมา
2. ควรเห็น:
   - Discord ได้ embed
   - Google Sheet มีแถวใหม่ (action, entry, SL, TP, ...)
3. ครั้งหน้ารัน **Read Lessons** จะอ่านแถวที่เพิ่งเพิ่ม → agents เห็น "ประวัติ"

## วิธี "สอน" agent ผ่าน Sheet

แต่ละแถวมีคอลัมน์ `outcome` กับ `lesson` ที่เริ่มเป็น `"pending"` / ว่าง

หลังเทรดจบ (ไม่ว่าผ่าน TP/SL หรือปิดเอง) — เปิด Sheet แก้:

| outcome | lesson |
|---------|--------|
| `win` | "EMA50 bounce setup ทำงานเมื่อ ATR > 15" |
| `loss` | "เข้าก่อน FOMC 2 ชม. — โดน whipsaw, ไม่ควรทำอีก" |
| `breakeven` | "ปิดเร็วเกิน อย่าตื่นตระหนกตอน RSI แตะ 70" |

ครั้งหน้ารัน workflow agents จะอ่าน lessons ทั้งหมด → ปรับการตัดสินใจ

นี่คือ **learning loop** ที่ทำให้ระบบดีขึ้นเรื่อย ๆ

## Model selection

Default ใน workflow:
- Bull/Bear/Quant: `claude-sonnet-4-6` (เร็วกว่า ถูกกว่า)
- Judge: `claude-opus-4-6` (ฉลาดสุดสำหรับการตัดสิน)

**อยากใช้ Opus 4.7 หมด?** แก้ใน node Anthropic Chat Model — แต่ Opus 4.7 ตัดทิ้ง `temperature` ดังนั้นถ้า n8n langchain version เก่ากว่า 1.3 จะ error

ถ้าเจอ error → คงไว้ที่ Sonnet 4.6 + Opus 4.6 ตามค่า default

## ค่าใช้จ่ายต่อ run

- 4 calls × Sonnet/Opus mix
- ครั้งแรก: ~$0.15-0.25
- รัน 2 ครั้ง/วัน × 22 วัน = **~$8-12/เดือน**

## Customization ที่นิยม

### เปลี่ยนสินทรัพย์

แก้ใน **Fetch Real Stock Data + Indicators** Code node:
- บรรทัดที่มี `GC=F` → เปลี่ยนเป็น `BTC-USD`, `EURUSD=X`, `^DJI`, ฯลฯ
- แล้วแก้ system prompts ของ agents ให้สอดคล้อง

### เพิ่มเงื่อนไข "ส่งเฉพาะ confidence สูง"

หลัง **Format Output** เพิ่ม **IF node**:
- Condition: `{{ $json.decision.confidence >= 0.7 }}`
- True → Discord
- False → Skip (แต่ยัง save Sheet)

### Multi-asset scan

ห่อ workflow ในลูปด้วย **Loop Over Items** node:
- Input: array of tickers `[{ticker: "GC=F"}, {ticker: "BTC-USD"}, ...]`
- รันทั้งหมด → ส่ง Discord summary 1 ตัว

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|--------|--------|---------|
| `Anthropic Chat Model` หาไม่เจอ | n8n เก่า | อัปเดต n8n ≥ 1.20 |
| `400 invalid_request_error` | sonnet ส่ง temperature ผิด | ลด typeVersion ของ Anthropic node เป็น 1.2 |
| Google Sheets 401 | OAuth หมดอายุ | re-authenticate credential |
| Output parser ไม่ทำงาน | Judge model ไม่ support tool use | เปลี่ยนเป็น Opus 4.6 หรือ Sonnet 4.5 |
| Agent วน loop | maxIterations ต่ำ | เพิ่ม `options.maxIterations: 5` |

## ⚠️ Disclaimer

```
- ข้อมูลนี้เพื่อการศึกษา ไม่ใช่คำแนะนำลงทุน
- AI Agent ตัดสินใจผิดได้ — ทดลองใน demo ก่อนเสมอ
- Yahoo Finance delay 15-20 นาที
- Sheets memory ดีก็ต่อเมื่อ 'lesson' ที่คุณเขียนคุณภาพดี
```
