# 🩹 Troubleshooting — ปัญหาที่เจอจริง + วิธีแก้

เอกสารรวมปัญหาที่เจอจริงในการใช้งานจริง พร้อมวิธีแก้แบบตรงจุด เรียงจาก "เจอบ่อยที่สุด" ลงมา

---

## 📑 สารบัญปัญหา

1. [ตัวละครไม่โผล่บน Telegram แต่โผล่บน CLI](#1-ตัวละครไม่โผล่บน-telegram-แต่โผล่บน-cli)
2. [Session เก่า (ก่อนติดตั้ง) ไม่ส่ง event](#2-session-เก่า-ก่อนติดตั้ง-ไม่ส่ง-event)
3. [Build พังบน Windows (prettier "No files matching")](#3-build-พังบน-windows-prettier-no-files-matching)
4. ["remote port forwarding failed" หรือ tunnel ตาย](#4-remote-port-forwarding-failed-หรือ-tunnel-ตาย)
5. [Token mismatch หลังรีสตาร์ท office](#5-token-mismatch-หลังรีสตาร์ท-office)
6. [เช็คว่า event มาถึงไหม](#6-เช็คว่า-event-มาถึงไหม)
7. [ตัวละคร "หายไป"](#7-ตัวละคร-หายไป)
8. [อยากเห็นตัวละครขยับชัดๆ](#8-อยากเห็นตัวละครขยับชัดๆ)

---

## 1) ตัวละครไม่โผล่บน Telegram แต่โผล่บน CLI

### อาการ

- รัน `hermes` ใน CLI → ตัวละครขยับใน office ✅
- คุยผ่าน Telegram bot → Hermes ตอบได้ แต่ตัวละคร **ไม่ขยับ** ❌

### สาเหตุ

คุณกำลังใช้ **plugin `pixel_observer`** (Python) แทนที่จะเป็น bridge ปัญหาคือ Hermes gateway/Telegram **ยิงแค่ shell hooks** ไม่ยิง plugin hooks ส่วน CLI โหลด plugin ได้ เลยดูเหมือน "ใช้ได้ครึ่งเดียว"

### วิธีแก

**เปลี่ยนไปใช้ bridge (shell hooks)** ไม่ใช่ plugin:

1. ลง shell hooks ใน `~/.hermes/config.yaml` (ดู `config/hermes-hooks.yaml.snippet` หรือ README หัวข้อติดตั้ง)
2. ตรวจว่ามีบรรทัด `hooks_auto_accept: true`
3. รีสตาร์ท gateway ทีเดียว (ดูข้อ 2)

> 📖 รายละเอียดเต็มที่: [`PLUGIN-VS-BRIDGE.md`](PLUGIN-VS-BRIDGE.md)

---

## 2) Session เก่า (ก่อนติดตั้ง) ไม่ส่ง event

### อาการ

หลังลง bridge/hooks เสร็จ session ที่ **กำลังรันอยู่ก่อนแล้ว** ไม่ขยับใน office แต่ session ใหม่ที่เริ่มหลังติดตั้งใช้ได้

### สาเหตุ

Hermes โหลด `~/.hermes/config.yaml` (รวม hooks) **ตอน start session เท่านั้น** session ที่รันอยู่ก่อนยังถือ config เก่าอยู่ใน memory → ไม่รู้จัก hooks ใหม่

### วิธีแก

รีสตาร์ท gateway/profile **ครั้งเดียว** เพื่อให้โหลด hooks ใหม่:

```bash
# ถ้าใช้ gateway แบบ watchdog
sudo systemctl restart hermes-gateway-watchdog

# หรือ restart profile ตามรูปแบบของคุณ (เช่น restart service ที่รัน profile)
```

หลังจากนี้ **session ใหม่ทุกอัน** จะส่ง event หมด (session เก่าที่ค้างอยู่ต้องรอให้จบก่อน ไม่งั้นรอ restart)

> 💡 ไม่ต้องรีบ — รีสตาร์ทครั้งเดียวพอ อย่าวนลูป restart เพราะ session ที่กำลังทำงานอาจหาย

---

## 3) Build พังบน Windows (prettier "No files matching")

### อาการ

ตอนรัน `npm run build` บน Windows ขึ้น error:

```
[error] No files matching the pattern "..."
```

จาก `prettier`

### สาเหตุ

เครื่องหมายคำพูด/การขยาย glob ของ prettier ทำงานต่างกันบน Windows shell (cmd/PowerShell) เทียบกับ bash ทำให้ pattern ที่เขียนสำหรับ Unix หาไฟล์ไม่เจอ

### วิธีแก

อ่านวิธีแก้แบบละเอียดที่:

```
fixes/generate-messages.windows-quote-fix.md
```

สรุปคือแก้ quoting ใน script ของ `package.json` (เช่น ใช้ double-quote หรือปรับ glob) ให้เข้ากับ Windows

> 💡 **แนะนำ:** ถ้าเป็นไปได้ ให้ build บนเครื่อง Linux/VPS ตรงๆ แล้วค่อย deploy — ลดปัญหา shell ได้เยอะ

---

## 4) "remote port forwarding failed" หรือ tunnel ตาย

### อาการ

ใช้ SSH reverse tunnel (`ssh -R ...`) เพื่อเอา office จาก VPS มาดูที่ laptop แล้วเจอ:

```
Warning: remote port forwarding failed for listen port 3100
```

หรือ tunnel ตายหลังผ่านไปสักพัก ตัวละครหยุดขยับ

### สาเหตุ

โปรเจกต์นี้ **ออกแบบมาให้รัน office บนเครื่องเดียวกับ Hermes (VPS)** ไม่ได้ออกแบบให้ใช้ tunnel เลย reverse tunnel (`-R`) มักชนกับพอร์ตที่ค้างอยู่บน VPS และไม่ stable

### วิธีแก

**ทางแรก (แนะนำ): อย่าใช้ tunnel เลย** — รัน office บน VPS เครื่องเดียวกับ Hermes แล้วเปิดดูที่ `http://<YOUR_VPS_IP>:3100` ตรงๆ (หรือ bind `0.0.0.0`)

**ทางสอง (ถ้าต้องดูจากเครื่องอื่น): ใช้ SSH FORWARD tunnel ไม่ใช่ reverse**

```bash
ssh -L 3100:127.0.0.1:3100 <user>@<YOUR_VPS_IP>
```

แล้วเปิด `http://127.0.0.1:3100` บน laptop — เป็น forward tunnel (ทางเดียวจาก laptop เข้า VPS) ซึ่ง stable กว่าและไม่มีปัญหา "port forwarding failed"

> ⚠️ ห้ามใช้ `ssh -R` กับโปรเจกต์นี้ — มันจะตายเอง

---

## 5) Token mismatch หลังรีสตาร์ท office

### อาการ

รีสตาร์ท `pixel-office` แล้วกลัวว่า token ใน `server.json` เปลี่ยน → bridge จะ POST ไม่ผ่าน

### วิธีแก

**ไม่ต้องทำอะไรเลย** ✅

Bridge (`pixel_agents_bridge.py`) **อ่าน `~/.pixel-agents/server.json` ใหม่ทุกครั้ง** ที่ถูกเรียก → มันจะเห็น token ใหม่ของ office โดยอัตโนมัติ ไม่ต้อง restart Hermes ไม่ต้องแก้ config

> 💡 นี่คือเหตุผลที่ discovery ใช้ไฟล์แทนค่าคงที่ — ทำให้ office กับ bridge ไม่ต้อง sync กันเอง

---

## 6) เช็คว่า event มาถึงไหม

เมื่อตัวละครไม่ขยับ ให้ไล่จาก "ต้นน้ำ" ลงมา:

### ขั้น A — ดูที่ bridge (Hermes ยิง hook มาถึงไหม)

```bash
tail -f ~/.hermes/pixel_agents_bridge.log
```

ถ้ามีบรรทัด event ปรากฏ → Hermes ยิง hook ถูกทาง ปัญหาอยู่หลัง bridge ถ้าว่างเปล่า → Hermes ไม่ยิง (กลับไปดูข้อ 1 และ 2)

### ขั้น B — ดูที่ office (Bridge POST มาถึงไหม)

```bash
tail -f ~/pixel-office.log
```

มองหาบรรทัดที่มีคำว่า `Hermes:` เช่น:

```
Hermes: on_session_start  profile=telegram
Hermes: pre_tool_call     profile=telegram tool=Bash
```

ถ้ามี → office รับ event แล้ว ปัญหาน่าจะอยู่ที่เบราว์เซอร์/WebSocket (ลองรีเฟรชหน้า) ถ้าว่าง → bridge POST ไม่ถึง (เช็ค token ใน `server.json`, พอร์ต, ว่า office รันอยู่ไหม)

### ขั้น C — ดูว่า office รันอยู่ไหม

```bash
sudo systemctl status pixel-office
curl http://127.0.0.1:3100/api/health
```

---

## 7) ตัวละคร "หายไป"

### อาการ

เปิด office แล้วไม่เห็นตัวละครที่คาดไว้ หรือเห็นแป๊บเดียวแล้วหาย

### สาเหตุ

ตัวละคร **ปรากฏตอน active และไปนั่งพักตอน idle** — ตอน idle มันไม่ได้ "หาย" แค่นั่งเฉยๆ (อาจอยู่นอกขอบจอหรือเล็กจนมองข้าม) นี่คือพฤติกรรมปกติไม่ใช่ bug

### วิธีแก

1. **รีเฟรชหน้าแรงๆ** — `Ctrl+F5` (hard refresh) เพื่อเคลียร์ WebSocket เก่า
2. **ส่งข้อความถึง Hermes ตอนที่เปิดหน้าอยู่** — พอมี activity ตัวละครจะขยับให้เห็น
3. ถ้ายังไม่เห็น → ไล่ log ตามข้อ 6

---

## 8) อยากเห็นตัวละครขยับชัดๆ

### อาการ

เปิดแล้วเห็นตัวละคร "แทบไม่ขยับ" งงว่ามันทำงานจริงไหม

### วิธีแก

ส่ง task ที่ทำให้ Hermes **เรียก tool หลายครั้ง** เช่น:

- "อ่านไฟล์ทั้งหมดในโฟลเดอร์ X แล้วสรุปให้หน่อย" (trigger Read หลายรอบ)
- "ตรวจสถานะระบบ 5 อย่างแล้วรายงาน" (trigger Bash หลายครั้ง)

หลักการ: **session ยาว = ตัวละครขยับนาน** เพราะทุก `pre_tool_call`/`post_tool_call` = หนึ่งการขยับ ถ้าส่ง task สั้นๆ ที่ตอบคำเดียว ตัวละครจะขยับนิดเดียวแล้วกลับไปนั่ง idle

> 💡 ทดลองส่ง task ยาวแล้วเปิด office ค้างไว้ — จะเห็นตัวละคร "ทำงานจริง" ชัดเจน

---

## 🆘 ยังไม่ได้ผล?

ถ้าลองครบทุกข้อแล้วยังติด:

1. เช็ค `sudo systemctl status pixel-office` — service รันอยู่ไหม
2. เช็ค `~/.hermes/config.yaml` — hooks อยู่ครบไหม, `hooks_auto_accept: true` ตั้งไว้ไหม
3. เช็ค `~/.pixel-agents/server.json` — มีอยู่ไหม, port ตรงกับที่ service รันไหม
4. ไล่ log ทั้งสอง (ข้อ 6)
5. เปิด issue บน GitHub พร้อมแปะ log จาก `~/.hermes/pixel_agents_bridge.log` และ `~/pixel-office.log`

---

กลับไป [README](../README.md) · [Architecture](ARCHITECTURE.md) · [Plugin vs Bridge](PLUGIN-VS-BRIDGE.md)
