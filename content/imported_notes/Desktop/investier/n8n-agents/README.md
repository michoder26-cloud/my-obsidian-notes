# Stock Analysis Multi-Agent (n8n)

ระบบวิเคราะห์หุ้น US + ETF ด้วย AI agent หลายตัว ส่งรายงานเข้า email ทุกเช้า

## ไฟล์ในโฟลเดอร์นี้

| ไฟล์ | ใช้ทำอะไร |
|------|----------|
| `workflow.json` | Import เข้า n8n ได้เลย |
| `PROMPTS.md` | Prompts แต่ละ agent (เผื่ออยากปรับ) |
| `email-template.html` | ตัวอย่าง email (เปิดใน browser ดูหน้าตา) |

## วิธี Import เข้า n8n

1. เปิด n8n
2. กด **Workflows** → **Import from File**
3. เลือก `workflow.json`
4. ตั้งค่า credentials:
   - **AI Agent nodes** → เชื่อม Anthropic / OpenAI ของคุณ
   - **Send Email node** → เชื่อม Gmail ของคุณ
5. กด **Save** แล้วกด **Active**

## โครงสร้าง Workflow

```
Schedule (8AM) 
  → Set Watchlist (AAPL, MSFT, NVDA, ...)
  → Agent 1: Data Collector (ที่คุณมีอยู่แล้ว)
  → Parse JSON
  → [Agent 2: Technical] + [Agent 3: Sentiment] (ขนาน)
  → Merge
  → Agent 4: Decision Maker → เขียน HTML
  → Send Email
```

## ปรับแต่งง่ายๆ

### เปลี่ยนหุ้นที่ติดตาม
แก้ที่ node **"Set Watchlist"** ใส่ ticker คั่นด้วย comma
```
AAPL,MSFT,NVDA,GOOGL,TSLA,SPY,QQQ,VOO
```

### เปลี่ยนเวลาส่ง
แก้ที่ node **"Schedule"** — cron `0 8 * * 1-5` = 8 โมงเช้า จันทร์-ศุกร์

### เปลี่ยน email ปลายทาง
แก้ที่ node **"Send Email"** ช่อง `sendTo`

## โมเดลที่แนะนำ

| Agent | Model | เหตุผล |
|-------|-------|--------|
| 1. Collector | Sonnet 4.6 | ใช้ tool ดึงข้อมูล |
| 2. Technical | Haiku 4.5 | คำนวณง่าย ใช้ตัวถูก |
| 3. Sentiment | Haiku 4.5 | อ่านข่าวสรุปสั้น |
| 4. Decision | **Opus 4.7** | ตัดสินใจ + เขียน HTML สวยๆ |

## ปัญหาที่อาจเจอ

**1. Parse JSON พัง**
- Agent ตอบเป็น text ปนกับ JSON
- แก้: เพิ่ม `"ตอบเฉพาะ JSON เท่านั้น"` ใน prompt

**2. Agent 1 ดึงข้อมูลไม่ครบ**
- เช็คว่า tool/API ที่ต่ออยู่ทำงานได้ไหม
- ถ้าใช้ Yahoo Finance ฟรี อาจโดน rate limit

**3. Email ไม่สวย**
- เช็คว่า Agent 4 ตอบเป็น HTML จริงๆ ไม่มี ` ```html ` ครอบ
- เปิด `email-template.html` ดูหน้าตาที่ควรจะเป็น

## ขั้นตอนถัดไป (ถ้าอยากต่อยอด)

- เพิ่ม Google Sheets node เก็บประวัติคำแนะนำทุกวัน
- เพิ่ม IF node ให้ส่งเฉพาะวันที่มีสัญญาณ Buy
- เพิ่ม Discord/Line notify นอกจาก email
