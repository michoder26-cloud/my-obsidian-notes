# 🧩 Plugin vs Bridge — บทเรียนที่ "เจ็บ" ที่สุด

> เอกสารนี้คือสรุปของปัญหาที่กินเวลานานที่สุดของโปรเจกต์นี้ — อ่านให้จบก่อนตัดสินใจเลือกแนวทาง

---

## TL;DR (อ่านบรรทัดเดียว)

> **ใช้ bridge (shell hooks) อย่าเด็ดขาดที่จะใช้ plugin `pixel_observer`** — plugin ใช้กับ CLI ได้ แต่ **Telegram/gateway ของ Hermes ไม่ยิง plugin hooks** เลยมักจะมองไม่เห็นตัวละครบน Telegram ถ้าใช้ plugin

---

## 📜 เกริ่น: pixel-agents มีสองทางเข้า

pixel-agents (upstream) ตอนแรกออกแบบมาให้ Claude Code โดยใช้ plugin ของ Python runtime (เช่น `pixel_observer`). เมื่อย้ายมา Hermes เรามีสองทางเลือก:

| ทางเลือก | กลไก | ใช้กับ CLI | ใช้กับ Telegram/gateway |
|---|---|---|---|
| **A. Plugin `pixel_observer`** (Python) | ลงทะเบียนเป็น plugin ของ Hermes Python runtime รับ `pre_tool_call`/`post_tool_call` hooks ของ plugin | ✅ ได้ | ❌ **ไม่ได้** |
| **B. Bridge script** (shell hooks) | ลง shell hooks ใน `~/.hermes/config.yaml` → Hermes spawn `pixel_agents_bridge.py` | ✅ ได้ | ✅ **ได้** |

---

## 🤒 อาการที่เจอ (ที่ทำให้เราเลือก bridge)

ตอนแรกเราลองทาง A (plugin) เพราะดู "สะอาดกว่า" — ไม่ต้องแตะ config.yaml ไม่ต้อง spawn process แต่แล้วเจออาการนี้:

### อาการ

- ✅ พอรัน **`hermes` ใน CLI** ทำงาน → ตัวละครโผล่ใน office ขยับได้ปกติ
- ❌ พอใช้ผ่าน **Telegram bot** หรือ **gateway** → ตัวละคร **ไม่ขยับเลย** ทั้งที่ Hermes ตอบข้อความได้

สรุปคือ: **CLI โผล่ แต่ Telegram ไม่โผล่**

---

## 🔬 สาเหตุ (หลังขุดลึก)

หลังจากไล่ดูโค้ดและ log พบว่า Hermes มี **"สองชุด hooks" ที่แยกจากกัน**:

### 1) Plugin hooks (ของ Python runtime)

ปลั๊กอิน `pixel_observer` ลงทะเบียนกับ **Python runtime ของ Hermes** เฉพาะตอนที่มีการสปาน CLI session (interactive) เท่านั้น ทำให้:

- ✅ CLI session → runtime เริ่ม → โหลด plugin → ยิง plugin hooks → ตัวละครขยับ
- ❌ Telegram/gateway → **ไม่ได้ผ่าน Python runtime แบบเดียวกัน** → plugin hooks ไม่ถูกยิง

### 2) Shell hooks (ของ `config.yaml`)

Shell hooks ที่ตั้งใน `~/.hermes/config.yaml` เป็น **กลไกระดับ core** ที่ Hermes ยิงทุก source ไม่ว่าจะมาจาก CLI, Telegram, gateway, หรือ cron เพราะมันเป็นส่วนหนึ่งของ lifecycle หลักของ session

> 💡 คิดง่ายๆ: **plugin = ส่วนเสริมที่ runtime บางตัวโหลด; shell hook = เสียงระฆังที่ Hermes เล่นทุกครั้ง**

### เหตุผลเสริมที่ทำให้ plugin ใช้ไม่ได้กับ Telegram

1. **gateway เริ่มก่อน enable plugin** — หลายครั้ง gateway/watchdog start ตอนบูตเครื่อง ก่อนที่ plugin จะถูก enable/ตรวจพบ → plugin ไม่อยู่ในใจ gateway ตั้งแต่ต้น
2. **gateway ไม่โหลด plugin hooks** — แม้จะ enable plugin ภายหลัง gateway ก็มักไม่ re-scan และไม่โหลด plugin hooks เข้ามาใน pipeline ของมัน → Telegram event ไม่วิ่งผ่าน plugin

ผลรวม: plugin จึง **"ทำงานเฉพาะ CLI"** เสมอในงานของเรา

---

## ✅ วิธีแก้ที่ใช้จริง: ใช้ bridge (shell hooks)

แทนที่จะสู้กับ lifecycle ของ plugin เราเลือกทางที่ Hermes รับประกันว่ายิงทุกที่:

1. ลง shell hooks ใน `~/.hermes/config.yaml` (ดู `config/hermes-hooks.yaml.snippet`)
2. ทุก hook เรียก `python3 ~/.hermes/pixel_agents_bridge.py`
3. Bridge แปลงและ POST ไป office (อ่านรายละเอียดใน `ARCHITECTURE.md`)

ผล:

- ✅ CLI → ขยับ
- ✅ Telegram → ขยับ
- ✅ gateway → ขยับ
- ✅ cron → ขยับ

ทั้งหมดเพราะ shell hook เป็นกลไกเดียวที่ Hermes ยิงจาก **ทุก source**

---

## 🧪 ถ้าอยากลอง plugin อยู่ดี (ทำได้ แต่ระวัง)

ถ้าใครอยากทดลองทาง plugin ก็ทำได้ — ไฟล์อยู่ที่ `hermes-plugin/pixel_observer` ของ upstream fork ขั้นตอนคร่าวๆ:

```bash
# (ตัวอย่าง — ปรับตาม upstream)
cd hermes-plugin/pixel_observer
# ติดตั้งตามวิธีของ Hermes plugin
hermes plugins install ./pixel_observer
# หรือตามกลไก enable plugin ของเวอร์ชัน Hermes ที่คุณใช้
```

**แต่จำไว้: มันจะไม่ทำงานกับ Telegram/gateway** (เหตุผลตามด้านบน) ใช้ได้แค่ตอนรัน `hermes` ใน CLI เท่านั้น

### เมื่อไรควรพิจารณา plugin?

- 🧪 ทดลอง/ศึกษาในเครื่องตัวเอง ไม่สน Telegram
- 🎯 อยากได้ข้อมูล field เฉพาะที่มีแค่ใน Python runtime (เช่น internal state)

ถ้าเป้าหมายคือ "ดู agent ทำงานจากทุกช่องทาง" — **bridge คือคำตอบเดียว**

---

## 📊 ตารางเปรียบเทียบสรุป

| มิติ | Plugin `pixel_observer` | Bridge (shell hooks) |
|---|---|---|
| CLI | ✅ | ✅ |
| Telegram | ❌ | ✅ |
| Gateway | ❌ | ✅ |
| Cron | ❌ (ขึ้น runtime) | ✅ |
| การตั้งค่า | enable plugin | merge hooks ใน `config.yaml` |
| Process overhead | ใน process เดียวกับ runtime | spawn python ทุก event (เล็กน้อย) |
| ความเสถียรข้าม source | ต่ำ | **สูง** |
| ข้อแนะนำ | เฉพาะทดลอง CLI | ✅ **ใช้งานจริง** |

---

## 🎯 บทสรุป

- **ปัญหา:** plugin pixel_observer ทำงานกับ CLI แต่ไม่ขยับบน Telegram
- **สาเหตุ:** Hermes gateway/Telegram ยิงแค่ shell hooks ไม่ยิง plugin hooks; gateway มัก start ก่อนที่ plugin จะถูก enable/โหลด
- **วิธีแก้:** ใช้ **bridge script** ที่ register เป็น **shell hooks** แทน — ได้รับการรับประกันจาก Hermes ว่าจะยิงจากทุก source

> อย่าเสียเวลาสู้กับ plugin ถ้าเป้าหมายคือดู Telegram — ใช้ bridge ตั้งแต่แรก ประหยัดเวลาหลายชั่วโมง

---

กลับไป [README](../README.md) · [Architecture](ARCHITECTURE.md) · [Troubleshooting](TROUBLESHOOTING.md)
