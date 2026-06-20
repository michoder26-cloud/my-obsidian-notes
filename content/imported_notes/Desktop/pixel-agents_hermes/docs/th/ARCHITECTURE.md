# 📐 สถาปัตยกรรมเชิงลึก — Hermes Pixel Agents

เอกสารนี้อธิบายว่าทุกชิ้นส่วนเชื่อมกันยังไง เหมาะสำหรับคนอยากแก้/ขยาย/เข้าใจสิ่งที่อยู่ใต้ฝาครอบ

---

## 🧩 ส่วนประกอบหลัก (4 ชิ้น)

```
┌──────────────┐   ┌────────────────┐   ┌────────────────────┐   ┌──────────────┐
│   Hermes     │──▶│  Bridge script │──▶│   pixel-office      │──▶│   Browser    │
│  (runtime)   │   │  (Python)      │   │  (Node + system  d) │   │  (pixel UI)  │
│              │   │                │   │  ├─ HermesBridge    │   │              │
│  shell hooks │   │  reads         │   │  └─ WebSocket /ws   │   │              │
│  in config   │   │  server.json   │   │                     │   │              │
└──────────────┘   └────────────────┘   └────────────────────┘   └──────────────┘
   ผู้ส่ง event       ผู้แปลง+ส่ง            ผู้รับ+กระจาย           ผู้แสดงผล
```

### 1) Hermes runtime (ฝั่งส่ง)

Hermes agent รันอยู่แล้วบนเครื่อง (VPS) เราเพิ่ม **shell hooks** เข้าไปใน `~/.hermes/config.yaml` ทุกครั้งที่เกิด event (`on_session_start`, `pre_tool_call`, …) Hermes จะ spawn คำสั่งที่เรากำหนด และส่ง payload (JSON) มาทาง stdin

> จุดสำคัญ: Hermes **gateway/Telegram ยิงแค่ shell hooks** — ไม่ยิง plugin hooks ของ Python runtime นี่คือเหตุผลที่ต้องมี bridge (อ่าน `PLUGIN-VS-BRIDGE.md`)

### 2) Bridge script (`pixel_agents_bridge.py`)

Python script เล็กๆ ที่ทำ 3 อย่าง:

1. **อ่าน event payload** จาก stdin ที่ Hermes ส่งมา
2. **ค้นหา office** โดยอ่าน `~/.pixel-agents/server.json` — ไฟล์นี้เก็บ `port` และ `token` ของ office (office เขียนตอน start)
3. **POST** ไป `http://127.0.0.1:<port>/api/hooks/hermes` พร้อม header Authorization ที่ใช้ token

Bridge **อ่าน `server.json` ใหม่ทุกครั้ง** ที่ถูกเรียก → แม้ office รีสตาร์ทแล้ว token เปลี่ยน ก็ไม่ต้องรีคอนฟิก

### 3) pixel-office (ฝั่งรับ + กระจาย)

pixel-agents fork ที่เราเพิ่ม `HermesBridge` เข้าไป รันเป็น **systemd service** ชื่อ `pixel-office`

- **ฟัง HTTP** ที่ `POST /api/hooks/hermes` — จุดรับ event จาก bridge (ต้องมี token ถูก)
- **ฟัง WebSocket** ที่ `/ws` — ช่องทางส่งข้อมูลไปยังเบราว์เซอร์ทุกบานที่เปิด office
- **เสิร์ฟหน้าเว็บ** pixel-art ที่ `/`

เมื่อ `HermesBridge` รับ event แล้ว มันแปลงเป็น message สำหรับ UI แล้ว broadcast ผ่าน `/ws` ไปทุก client

### 4) Browser (ฝั่งแสดงผล)

เบราว์เซอร์เปิด `http://<VPS>:3100` แล้วต่อ WebSocket เข้า `/ws` ทุก message ที่วิ่งมาจะขับเคลื่อนตัวละคร pixel (เปลี่ยนท่า/ย้ายจุด/เปลี่ยนอิโมติคอน)

---

## 🆔 ทำไมตัวละครถึง "1 ตัวต่อ profile" — เรื่อง uuid5

แต่ละตัวละครต้องมี **ID คงที่ตลอดอายุการใช้งาน** ไม่ใช่งอกใหม่ทุกข้อความ ไม่งั้น office จะเต็มไปด้วยตัวซ้ำ

เราใช้ **uuid5** (UUID แบบกำหนดโดย hash ของชื่อ + namespace):

```python
character_id = uuid5(NAMESPACE, profile_name)
# เช่น profile "telegram" → ID เดิมเสมอ ไม่ว่าจะส่งกี่ข้อความ
```

ผล:

- ✅ **Persistent** — profile `trader` คือตัวละครตัวเดิมเสมอ แม้รีสตาร์ท office หรือ VPS
- ✅ **Stable** — ตัวละคร "นั่งรอ" อยู่ก่อน พอมี activity จึงขยับ; ไม่ใช่เกิด-ตายทุก event
- ✅ **Multi-profile** — `telegram`, `trader`, `default`, … แต่ละ profile ได้ตัวการ์ตูนตัวเอง อยู่ใน office เดียวกันได้พร้อมกัน
- ✅ **Subagent** ก็ใช้หลักเดียวกัน — subagent ของ profile นั้นๆ มี ID ของมัน ปรากฏ/ออกตอน `subagent_start`/`subagent_stop`

> ถ้าคุณเปลี่ยนชื่อ profile → ID เปลี่ยน → ถือว่าเป็นตัวละครใหม่ (ตัวเก่ายัง "นั่ง" อยู่ใน office จนกว่าจะ idle timeout)

---

## 🔍 Discovery — bridge หา office ยังไง

office ไม่ได้เขียน port ตายตัว เพราะอาจชนกับ service อื่น มันเลยเขียน "นามบัตร" ลงไฟล์ตอน start:

**`~/.pixel-agents/server.json`**

```json
{
  "port": 3100,
  "token": "<random-token-written-at-startup>",
  "host": "127.0.0.1"
}
```

Bridge อ่านไฟล์นี้ทุกครั้ง → รู้ทั้ง port และ token → POST ได้ถูกที่ถูกัว

> 💡 ข้อดี: ถ้าคุณเปลี่ยน port ใน service หรือ office รีสตาร์ทแล้วออก random token ใหม่ **bridge จะตามทันเอง** ไม่ต้องแก้ config ไม่ต้อง restart Hermes

---

## 🤔 ทำไมใช้ `/api/hooks/hermes` ไม่ใช่ `/api/hooks/claude`

pixel-agents ต้นน้ำ (upstream) ออกแบบมาให้ Claude Code ตั้งแต่แรก — มี endpoint `/api/hooks/claude` ที่ map กับ Claude runtime path เฉพาะ (เช่น `SessionStart`, `PreToolUse` ที่มีโครงสร้าง field เฉพาะของ Claude)

เราลองใช้ Claude path กับ Hermes แล้ว **render ไม่ดี** เพราะ:

| ปัญหา | รายละเอียด |
|---|---|
| 🧩 Schema ไม่ตรง | field ของ Hermes event ต่างจาก Claude (ชื่อ tool, โครงสร้าง session) → ตัวละครเดี้ยง/ขยับผิดจังหวะ |
| 🎭 ไม่มี Telegram/gateway | Claude path ออกแบบตาม Claude CLI เท่านั้น ไม่มีมุมมองสำหรับหลาย source (Telegram, cron) |
| 🔁 ตัวละครซ้ำซ้อน | Claude path ใช้ logic ID ที่ไม่ตรงกับแนวคิด "profile" ของ Hermes → ตัวละครเดียวกันอาจถูกสร้างหลายตัว |

เลยตัดสินใจ **เพิ่ม endpoint ใหม่ `/api/hooks/hermes`** พร้อม `HermesBridge` ที่ออกแบบมาเฉพาะสำหรับ event schema ของ Hermes:

- ✅ Map field Hermes → action ใน office ได้ถูกต้อง
- ✅ ใช้ uuid5 per-profile → ตัวละครเสถียร
- ✅ รองรับหลาย source (CLI/Telegram/gateway/cron) เพราะทั้งหมดยิง shell hook เดียวกัน

> upstream Claude path ยังอยู่ครบ (ไม่ได้ลบ) — ใครใช้ Claude Code ก็ยังใช้ pixel-agents ได้ตามปกติ เราแค่เพิ่มทางเลือกสำหรับ Hermes

---

## 🔄 วงจรชีวิตของ event (End-to-End)

ลองตามดู event `pre_tool_call` ตั้งแต่เกิดถึงหน้าจอ:

```
1. Hermes กำลังจะเรียก tool (เช่น Bash)
        │
        ▼
2. Hermes ยิง shell hook "pre_tool_call"
   → spawn: python3 ~/.hermes/pixel_agents_bridge.py
   → ส่ง JSON payload ทาง stdin
   { "hook": "pre_tool_call", "profile": "telegram",
     "tool": "Bash", "session_id": "abc123", ... }
        │
        ▼
3. Bridge อ่าน stdin
   → อ่าน ~/.pixel-agents/server.json  (ได้ port=3100, token=<...>)
   → แปลง payload เป็นรูปแบบ HermesBridge
   → POST http://127.0.0.1:3100/api/hooks/hermes
      Authorization: Bearer <token>
        │
        ▼
4. pixel-office รับที่ HermesBridge
   → ตรวจ token ✓
   → คำนวณ character_id = uuid5(NAMESPACE, "telegram")
   → สร้าง message: { character_id, action: "use_tool", tool: "Bash", ... }
   → broadcast ผ่าน WebSocket /ws
        │
        ▼
5. เบราว์เซอร์ทุกบานได้รับ message
   → เอนจิน pixel ขับตัวละคร "telegram" ไปที่เครื่อง Bash
   → เปลี่ยน animation เป็น "พิมพ์/ทำงาน"
        │
        ▼
6. (ต่อมา) post_tool_call มา → ตัวละคร เปลี่ยนท่า "เสร็จแล้ว"
```

ทั้งหมดนี้ใช้เวลา < 100ms เพราะวิ่ง localhost

---

## 📊 สรุปตาราง endpoint

| Endpoint | Method | ใครเรียก | สิ่งที่ทำ |
|---|---|---|---|
| `/` | GET | เบราว์เซอร์ | เสิร์ฟหน้า office pixel-art |
| `/ws` | WebSocket | เบราว์เซอร์ | รับ push event แบบ real-time |
| `/api/hooks/hermes` | POST (ต้องมี token) | bridge script | รับ Hermes event → broadcast |
| `/api/hooks/claude` | POST | Claude Code (upstream) | รับ Claude event (มีอยู่แต่ไม่ใช้กับ Hermes) |
| `/api/health` | GET | ใครก็ได้ | health check (200 OK) |

---

## 🔧 จุดที่อาจอยากขยาย

- **เพิ่ม event type ใหม่:** แก้ bridge ให้ส่ง field เพิ่ม แล้วแก้ `server/src/hermesBridge.ts` ให้ map เป็น action ใหม่
- **เปลี่ยน namespace uuid:** แก้ค่าคงที่ใน bridge และ `hermesBridge.ts` (ทำพร้อมกัน ไม่งั้น ID จะไม่ตรง)
- **รองรับ multi-machine:** ปัจจุบันออกแบบให้รันบนเครื่องเดียว ถ้าอยากแยก Hermes กับ office คนละเครื่อง ต้องเปลี่ยน bridge ให้ POST ข้ามเครือข่าย (เปิด port + จัดการ token/HTTPS เอง)

---

กลับไป [README](../README.md) · [Plugin vs Bridge](PLUGIN-VS-BRIDGE.md) · [Troubleshooting](TROUBLESHOOTING.md)
