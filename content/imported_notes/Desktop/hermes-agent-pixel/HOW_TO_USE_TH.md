# Pixel Agent Office สำหรับ Hermes — วิธีใช้

ดู Hermes agent ของคุณทำงานแบบ real-time เป็นตัวละครใน office สไตล์ pixel-art
(ใช้ bridge จาก [aiunlocked1412/hermes-agent-pixel](https://github.com/aiunlocked1412/hermes-agent-pixel))

---

## ✅ การตั้งค่าสุดท้าย (ใช้งานจริง) — office รันบน VPS

**เปิดดู:** 👉 **http://13.140.183.183:3100**

Office รันบน VPS (ข้าง Hermes) เป็น **systemd service `pixel-office.service`**:
- ไม่ต้องใช้ SSH tunnel (event วิ่ง localhost บน VPS → แข็งแกร่ง ไม่ตาย)
- auto-restart + รันตอนบู๊ต VPS อัตโนมัติ
- ไม่ต้องมี API key (Hermes บน VPS authenticated อยู่แล้ว)
- **Telegram, cron, CLI** ทุก session ส่งเข้า office หมด → ตัวละครตัวละคร **1 ตัวต่อ profile** (trader / telegram / default …)

**คุณแค่:** เปิด http://13.140.183.183:3100 → ใช้ Hermes ปกติ (Telegram ฯลฯ) → ดูตัวละคร

**จัดการ service (บน VPS):**
```
ssh -i C:\Users\TKTF\Desktop\VPS1\id_rsa root@13.140.183.183 "systemctl status pixel-office"
ssh -i C:\Users\TKTF\Desktop\VPS1\id_rsa root@13.140.183.183 "systemctl restart pixel-office"
tail ล็อก: ssh ... "tail -30 /root/pixel-office.log"
```

**ส่วนประกอบบน VPS:**
- Office: `/root/hermes-agent-pixel/pixel-agents` (build แล้ว) → systemd `pixel-office.service`
- Bridge: `/root/.hermes/pixel_agents_bridge.py` (แก้ให้ส่ง `/api/hooks/hermes`; เดิมสำรองไว้ `.v1bak`)
- plugin `pixel_observer`: **ปิดแล้ว** (bridge ครอบคลุมทุก session อยู่แล้ว)

---

> ℹ️ ด้านล่างเป็นวิธีเก่า (local office + SSH tunnel) ที่**เลิกใช้แล้ว** เก็บไว้อ้างอิง
> office บน VPS ด้านบนแทนที่หมดแล้ว

## สถานะปัจจุบัน (ทำเสร็จแล้ว ✅)

| ส่วน | สถานะ |
|---|---|
| Office (มี Hermes bridge) | build เสร็จ, รันบน **http://127.0.0.1:3100** |
| Hermes plugin `pixel_observer` | ติดตั้ง + **enable แล้ว** |
| Bridge | ทดสอบผ่าน — event ของ Hermes → ตัวละครปรากฏ + ขยับจริง |
| VS Code extension เดิม + `~/.claude/settings.json` | **ไม่ถูกแตะ** (แยก home dir อิสระ) |

## ✅ Mode B — VPS Hermes (ใช้ได้เลย ไม่ต้องมี API key) ★แนะนำ

Hermes บน VPS (`13.140.183.183`) **ใช้ได้แล้ว** (model `kimi-k2.6` ผ่าน Ollama Cloud + key OpenRouter/GLM) ส่วนเครื่อง local ไม่มี key เลยใช้ตัว VPS โดยส่ง events เข้า office ผ่าน SSH reverse tunnel (ทดสอบผ่านจริง — ตัวละครของ VPS Hermes โผล่ใน office และขยับ)

**เปิดใช้แบบคลิกเดียว:** ดับเบิ้ลคลิก **`start_vps_bridge.bat`**
- เริ่ม office local (:3100) + sync token ไป VPS + เปิด reverse tunnel
- แล้ว Hermes session ใดๆ บน VPS จะโผล่ใน http://127.0.0.1:3100

**วิธีทดสอบด่วน** (รันในเทอร์มินัล):
```
ssh -i C:\Users\TKTF\Desktop\VPS1\id_rsa root@13.140.183.183 "hermes --cli --yolo -z 'Reply OK'"
```
→ ตัวละคร `cli` โผล่ใน office

**ใช้ผ่านหน้าเว็บ (dashboard 9119):** เปิด http://13.140.183.183:9119 → แชต์ → session โผล่ใน office
> ถ้า session จากหน้าเว็บไม่โผล่ (dashboard รันมาก่อนตอนยังไม่ enable plugin) ให้ restart ครั้งเดียว:
> `ssh -i C:\Users\TKTF\Desktop\VPS1\id_rsa root@13.140.183.183 "systemctl restart hermes-gateway-watchdog"`
> (มี watchdog ดูแล restart ให้อัตโนมัติ ปลอดภัย)

---

## Mode A — Local Hermes (ต้องมี API key เอง)

## วิธีเปิดใช้ (ทุกครั้ง)

1. ดับเบิลคลิก **`start_pixel_office.bat`** (หรือรันใน Terminal)
   - เปิด office ในเบราว์เซอร์ (http://127.0.0.1:3100)
   - เปิดหน้าต่าง cmd ที่ตั้งค่า env (`PIXEL_AGENTS_URL`/`TOKEN`) ให้แล้ว
2. ในหน้าต่าง cmd นั้น พิมพ์:
   ```
   hermes
   ```
3. ส่งข้อความใน Hermes → ตัวละครจะปรากฏใน office และขยับ (พิมพ์/อ่าน/รอ/แบ่งงาน subagent) ตามที่มันทำ

## ⚠️ ขั้นตอนเดียวที่ขาด (ทำครั้งเดียว): ตั้งค่า provider ให้ Hermes

ตอนนี้ Hermes ยัง **ไม่มี LLM provider** (ไม่มี API key) → รัน session ไม่ได้ ทำอย่างใดอย่างหนึ่ง:

- **ง่ายสุด:** พิมพ์ `hermes model` แล้วเลือก provider/model (เช่น Nous Portal — login ครั้งเดียว)
- **หรือ** เปิด `C:\Users\TKTF\AppData\Local\hermes\.env` แล้วเปิดบรรทัดใดบรรทัดหนึ่ง ใส่ key ของคุณ เช่น:
  ```
  OPENROUTER_API_KEY=sk-or-v1-xxxxxxxx
  ```
  (แล้วเซ็ต `LLM_MODEL` ให้ตรงกับ model ที่จะใช้)

หลังตั้งค่าแล้ว รัน `hermes` ในหน้าต่างที่ launcher เปิดให้ → จะเห็นตัวละครทันที

## สถาปัตยกรรม / ทำไมถึงแยก home dir

- VS Code Pixel Agents extension รัน server ของมันอยู่บน port 51883 (ในโปรเซส VS Code เอง) ซึ่ง **ไม่มี Hermes bridge**
- ถ้าเรารัน standalone ปกติ มันจะ "ฝังตัว"ตาม server ของ extension แทนเปิดของตัวเอง → bridge ไม่ทำงาน
- ทางแก้: รัน office ของเราด้วย `USERPROFILE` แยกต่างหาก (`_office_home/`) → มันเปิด server ของมันเองบน 3100 พร้อม bridge โดยไม่กระทบ extension เดิมเลย
- Hermes plugin ชี้ไป office เราผ่าน env var (token เปลี่ยนทุกครั้งที่รับ — launcher อ่านให้อัตโนมัติ)

## ไฟล์สำคัญ

- `pixel-agents/` — fork ที่มี bridge (`server/src/hermesBridge.ts`)
- `hermes-plugin/pixel_observer/` — plugin (ติดตั้งไปที่ `AppData\Local\hermes\plugins\` แล้ว)
- `start_pixel_office.bat` / `.ps1` — launcher
- `_office_home/` — home แยกของ office (server.json, .claude ของมันเอง)
- `_backups_*/` — สำรอง server.json + claude settings ก่อนเริ่ม (ไว้ย้อนได้)

## ตรวจสอบ / แก้ปัญหา

- ตรวจ office: เปิด http://127.0.0.1:3100/api/health → ควรเห็น `{"status":"ok",...}`
- ยืนยัน plugin: `hermes plugins list` → pixel_observer ต้องเป็น **enabled**
- ทดสอบ bridge โดยไม่ใช้ Hermes: `cd pixel-agents && PA_TOKEN=<token> node verify_bridge.js`
  (`<token>` อ่านจาก `_office_home/.pixel-agents/server.json`)

## ย้อนกลับ / ลบ

- ปิด office: ปิดหน้าต่าง "Pixel Agents Office" หรือ kill node บน port 3100
- ปิด plugin: `hermes plugins disable pixel_observer`
- extension/Claude ตัวจริงไม่ถูกแตะอยู่แล้ว ไม่ต้องคืนค่าอะไร
