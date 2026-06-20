# 🏢 Hermes Pixel Agents — ดู Hermes agent ทำงานสดๆ เป็นตัวการ์ตูนใน office

> **เหมือนเกม The Sims แต่เป็นของจริง** — โปรเจกต์นี้ทำให้คุณเห็น Hermes AI agent ของคุณ "เดิน นั่ง พิมพ์ คุย" อยู่ใน office pixel-art ตรงหน้าจอเบราว์เซอร์ ขณะที่มันกำลังทำงานจริงๆ (รัน tool, ตอบ Telegram, ทำ cron job).

ทุกครั้งที่ Hermes agent เริ่ม session, เรียก tool, หรือจบงาน — ตัวละคร pixel ที่เป็นตัวแทนของมันจะ **ขยับและทำกิจกรรมใน office** แบบเรียลไทม์ คุณแค่เปิดเบราว์เซอร์แล้วนั่งดู เหมือนดูพนักงานทำงานผ่านกระจก

โปรเจกต์นี้ **เชื่อมสองโลกเข้าด้วยกัน**:

- [**NousResearch/hermes-agent**](https://github.com/NousResearch/hermes-agent) (MIT) — runtime ตัวจริงของ Hermes agent ที่รันคำสั่ง, คุยผ่าน Telegram/gateway/CLI/cron
- [**pixel-agents-hq/pixel-agents**](https://github.com/pixel-agents-hq/pixel-agents) — เอนจิน office สไตล์ pixel-art
- **ตัวเชื่อม (bridge):** เราเพิ่ม `HermesBridge` เข้าไปใน fork ของ pixel-agents + สคริปต์ `pixel_agents_bridge.py` ที่แปลง Hermes shell-hook events เป็นภาพเคลื่อนไหว

ผลลัพธ์: **ทุก session ของ Hermes — CLI, Telegram bot, gateway, cron — จะปรากฏเป็นตัวละครที่ขยับใน office เดียวกันหมด**

---

## 📑 สารบัญ

- [มันทำงานยังไง](#-มันทำงานยังไง)
- [สิ่งที่ต้องมีก่อน](#-สิ่งที่ต้องมีก่อน)
- [ติดตั้งใน 6 ขั้น](#-ติดตั้งใน-6-ขั้น)
- [เปิดดู office](#-เปิดดู-office)
- [จัดการ service](#-จัดการ-service)
- [ตารางไฟล์ใน repo](#-ตารางไฟล์ใน-repo)
- [ความปลอดภัย](#-ความปลอดภัย)
- [เอกสารเพิ่มเติม](#-เอกสารเพิ่มเติม)
- [Credits & License](#-credits--license)

---

## 🧠 มันทำงานยยังไง

ภาพรวม — ทุกอย่างวิ่งผ่าน **localhost** เพราะ office กับ Hermes รันอยู่บนเครื่องเดียวกัน (ปกติคือ VPS เครื่องเดียว) ไม่ต้องเปิด tunnel ไม่ต้องมี API key:

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                        เครื่อง VPS (ตัวเดียว)                       │
 │                                                                  │
 │   ┌───────────────┐    shell hooks        ┌──────────────────┐  │
 │   │  Hermes agent │ ───────────────────▶  │ pixel_agents_     │  │
 │   │  (CLI / TG /  │   (config.yaml)       │ bridge.py         │  │
 │   │   gateway /   │                       │  → HTTP POST      │  │
 │   │   cron)       │                       └────────┬──────────┘  │
 │   └───────────────┘                                │             │
 │                                                    ▼             │
 │                              POST http://127.0.0.1:3100/api/hooks/hermes
 │                                                    │             │
 │                                                    ▼             │
 │                                          ┌──────────────────┐    │
 │                                          │  pixel-office     │    │
 │                                          │  (systemd)        │    │
 │                                          │  ├─ HermesBridge   │    │
 │                                          │  └─ WebSocket /ws  │    │
 │                                          └────────┬──────────┘    │
 └───────────────────────────────────────────────────┼──────────────┘
                                                     │ WebSocket
                                                     ▼
                                          ┌──────────────────────┐
                                          │  เบราว์เซอร์คุณ         │
                                          │  หน้า office pixel    │
                                          │  http://<VPS>:3100     │
                                          └──────────────────────┘
```

**ไล่ลำดับง่ายๆ:**

1. Hermes agent เริ่มทำงาน (มีคนส่งข้อความใน Telegram / รัน CLI / ถึงเวลา cron)
2. Hermes ยิง **shell hook** ที่เราลงไว้ใน `~/.hermes/config.yaml` (เช่น `on_session_start`, `pre_tool_call`)
3. Shell hook เรียก `python3 ~/.hermes/pixel_agents_bridge.py` พร้อมส่ง event payload มาทาง stdin/argv
4. Bridge อ่าน `~/.pixel-agents/server.json` เพื่อหา port+token ของ office, แปลง event เป็น payload มาตรฐาน, แล้ว **POST** ไปที่ `http://127.0.0.1:3100/api/hooks/hermes`
5. ฝั่ง office `HermesBridge` รับ payload → ส่งผ่าน **WebSocket** ไปหาเบราว์เซอร์ทุกบานที่เปิดอยู่
6. ตัวละครใน office **ขยับ/เปลี่ยนอิโมติคอน/ย้ายจุด** ตามชนิดของ event

> 💡 **จุดสำคัญ:** ทั้งหมดวิ่งบน `127.0.0.1` (localhost) ของเครื่องเดียว — **ไม่ต้องเปิด SSH tunnel, ไม่ต้องมี API key, ไม่ต้องเสียบข้อมูลอะไร** เพราะ office กับ Hermes อยู่ด้วยกัน

---

## ✅ สิ่งที่ต้องมีก่อน

ก่อนเริ่ม ให้เช็คให้ครบ:

| สิ่งที่ต้องมี | หมายเหตุ |
|---|---|
| 🖥️ **เครื่องที่รัน Hermes อยู่แล้ว** | แนะนำ **VPS** (Ubuntu/Debian) เพราะ office ต้องรัน 24/7 ควบคู่กับ Hermes |
| 🔑 **สิทธิ์ root หรือ sudo** | ต้องใช้ติดตั้ง systemd service |
| 🟢 **Node.js 20+** | รันคำสั่ง `node --version` เช็ค; ถ้ายังไม่มีให้ลง (เช่นผ่าน nvm หรือ NodeSource) |
| 📦 **git** | `git --version` |
| 🤖 **Hermes ติดตั้งและใช้งานได้แล้ว** | ต้องมี `~/.hermes/config.yaml` และตั้ง provider/model ครบ (ลอง `hermes` แล้วตอบได้) |
| 🌐 **พอร์ต 3100** (หรือพอร์ตที่เลือก) เปิดบน VPS | ถ้าจะดูจากเบราว์เซอร์นอกเครื่อง — หรือใช้ SSH forward (ดูหัวข้อความปลอดภัย) |

> ⚠️ ถ้า Hermes ยังไม่ได้ลง ให้ลง Hermes ก่อนตามเอกสารของ [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) — โปรเจกต์นี้คาดหวังว่า Hermes รันได้แล้ว

---

## 🚀 ติดตั้งใน 6 ขั้น

มีสองทาง: **(A) รันสคริปต์เดียวจบ** สำหรับคนอยากเร็ว หรือ **(B) ทำมือทีละขั้น** สำหรับคนอยากเข้าใจ

### ทาง A — ติดตั้งด้วยคำสั่งเดียว (แนะนำ)

```bash
git clone https://github.com/aiunlocked1412/hermes-agent-pixel.git
cd hermes-agent-pixel
bash scripts/install.sh
```

สคริปต์นี้ทำทั้งหมดนี้ให้อัตโนมัติ:

1. **Clone/build** pixel-agents fork (ที่มี `HermesBridge`) ลงในตำแหน่งที่กำหนด
2. **Build** office (Next.js/Node) ให้พร้อมรัน
3. **ติดตั้ง bridge** (`pixel_agents_bridge.py`) ไปที่ `~/.hermes/pixel_agents_bridge.py`
4. **ลง shell hooks** ลงใน `~/.hermes/config.yaml` ของคุณ (merge อัตโนมัติ, ไม่ทลางไฟล์เดิม)
5. **ลง systemd service** (`pixel-office.service`) เพื่อให้ office รันเป็น background service ตื่นขึ้นมาเองทุกครั้งที่บูตเครื่อง
6. **Health check** — ยิง GET ไปที่ office แล้วรายงานว่าใช้ได้ไหม

เสร็จแล้วข้ามไป [เปิดดู office](#-เปิดดู-office) ได้เลย

---

### ทาง B — ทำมือทีละขั้น (สำหรับคนอยากควบคุม)

ถ้าไม่อยากวางใจสคริปต์ หรือเครื่องมี setup พิเศษ ทำตามนี้:

#### ขั้น 1 — Clone pixel-agents fork และ build

```bash
git clone https://github.com/aiunlocked1412/hermes-agent-pixel.git
cd hermes-agent-pixel
npm install
npm run build
```

#### ขั้น 2 — ติดตั้ง bridge script

คัดลอก `pixel_agents_bridge.py` ไปยังโฮมของ Hermes:

```bash
cp pixel_agents_bridge.py ~/.hermes/pixel_agents_bridge.py
chmod +x ~/.hermes/pixel_agents_bridge.py
```

ไฟล์นี้รับ event จาก Hermes shell hook ผ่าน stdin, อ่าน `~/.pixel-agents/server.json` (ที่ office เขียนไว้ตอน start) เพื่อหา `port` และ `token`, แล้ว POST ไป `/api/hooks/hermes`

#### ขั้น 3 — เพิ่ม shell hooks ลงใน `~/.hermes/config.yaml`

ไฟล์ตัวอย่างอยู่ที่ `config/hermes-hooks.yaml.snippet` ใน repo เปิด `~/.hermes/config.yaml` ของคุณแล้ว **merge** บล็อก `hooks:` ต่อไปนี้เข้าไป (ถ้ามี `hooks:` อยู่แล้วให้รวม key เดียวกัน อย่าลบของเก่า):

```yaml
hooks:
  hooks_auto_accept: true
  on_session_start:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  on_session_end:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  pre_tool_call:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  post_tool_call:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  subagent_start:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
  subagent_stop:
    - command: python3 ~/.hermes/pixel_agents_bridge.py
```

อธิบายแต่ละ hook:

| Hook | เมื่อไรยิง | ผลใน office |
|---|---|---|
| `on_session_start` | agent เริ่ม session | ตัวละคร "ตื่น"/เข้ามาใน office |
| `on_session_end` | agent จบ session | ตัวละคร ไปนั่งพัก/idle |
| `pre_tool_call` | ก่อนเรียก tool (Bash, Read, …) | ตัวละคร ขยับไปที่เครื่อง/โต๊ะ |
| `post_tool_call` | หลัง tool คืนผล | ตัวละคร เปลี่ยนท่า/อิโมติคอน |
| `subagent_start` | สปาน subagent | ตัวละคร subagent ปรากฏ |
| `subagent_stop` | subagent จบ | ตัวละคร subagent ออก |

> 💡 `hooks_auto_accept: true` สำคัญมาก — ถ้าไม่ตั้ง Hermes จะถามยืนยันทุกครั้งก่อนยิง hook แล้ว office จะกระตุก

#### ขั้น 4 — ลง systemd service

คัดลอก `pixel-office.service` ไปยัง systemd:

```bash
sudo cp pixel-office.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now pixel-office
```

service นี้รัน office (Node) บนพอร์ต 3100 และเขียน log ที่ `~/pixel-office.log`

#### ขั้น 5 — Health check

```bash
curl http://127.0.0.1:3100/api/health
# ควรได้ 200 OK
```

#### ขั้น 6 — Restart Hermes ทีเดียว (สำคัญ)

session ที่เริ่มก่อนลง hook **จะไม่ส่ง event** (เพราะ config โหลดตอน start) รีสตาร์ท gateway/profile ครั้งเดียวเพื่อโหลด hook ใหม่:

```bash
sudo systemctl restart hermes-gateway-watchdog   # หรือ restart profile ตามรูปแบบของคุณ
```

หลังจากนี้ session ใหม่ทุกอันจะส่ง event หมด

---

## 👀 เปิดดู office

เปิดเบราว์เซอร์ไปที่:

```
http://<YOUR_VPS_IP>:3100
```

แล้วใช้ Hermes ตามปกติ — ส่งข้อความใน Telegram, รัน CLI, หรือรอ cron ตัวละครจะเริ่มขยับ

### กฎเกณฑ์ของตัวละคร (สำคัญ — อ่านก่อนตกใจ)

- 🧍 **1 ตัวละคร ต่อ 1 profile** — แต่ละ Hermes profile (`trader`, `telegram`, `default`, …) คือตัวการ์ตูน **ตัวเดียวตลอดอายุการใช้งาน** ไม่ใช่เกิดใหม่ทุกข้อความ
- 🪑 **ตัวละคร "นั่งอยู่ก่อน"** — พอมี activity (เรียก tool / ตอบข้อความ) มันจึงจะขยับ; ตอน idle มันนั่งเฉยๆ ไม่ใช่หายไป
- 🔄 **persistent** — ID ของตัวละครคำนวณจากชื่อ profile (uuid5) เลยเหมือนตัวเดิมเสมอแม้รีสตาร์ท office
- 🎭 **subagent** มีตัวละครของมันเอง จะปรากฏ/ออกตอน `subagent_start`/`subagent_stop`

> ถ้าเปิดแล้ว "ไม่เห็นอะไรขยับ" — อย่าเพิ่งตกใจ ตัวละครอาจกำลัง idle อยู่ ลองส่ง task ที่ทำให้ Hermes เรียก tool หลายๆ ครั้ง แล้วดูอีกครั้ง

---

## 🛠️ จัดการ service

office รันเป็น systemd service ชื่อ `pixel-office`:

```bash
# ดูสถานะ
sudo systemctl status pixel-office

# รีสตาร์ท
sudo systemctl restart pixel-office

# หยุด
sudo systemctl stop pixel-office

# ดู log สดๆ
tail -f ~/pixel-office.log
```

### เปลี่ยน port / host

แก้ `/etc/systemd/system/pixel-office.service` — หาบรรทัดที่ตั้ง `PORT` หรือ `HOST` (ในส่วน `Environment=`) เช่น:

```ini
Environment=PORT=3200
Environment=HOST=127.0.0.1
```

แล้วรีโหลด:

```bash
sudo systemctl daemon-reload
sudo systemctl restart pixel-office
```

> ถ้าเปลี่ยน port อย่าลืมแก้ URL ใน `~/.hermes/pixel_agents_bridge.py` (หรือตั้ง `PIXEL_OFFICE_PORT`) และ URL ในเบราว์เซอร์ด้วย — แต่โดยปกติ bridge อ่าน port จาก `server.json` อัตโนมัติ

---

## 📁 ตารางไฟล์ใน repo

| ไฟล์ / โฟลเดอร์ | หน้าที่ |
|---|---|
| `scripts/install.sh` | สคริปต์ติดตั้งอัตโนมัติ (ทาง A) |
| `pixel_agents_bridge.py` | **สคริปต์หัวใจ** — รับ Hermes shell-hook event แล้ว POST ไป office |
| `config/hermes-hooks.yaml.snippet` | ตัวอย่างบล็อก `hooks:` ที่ต้อง merge เข้า `~/.hermes/config.yaml` |
| `pixel-office.service` | ไฟล์ systemd unit สำหรับรัน office เป็น service |
| `server/` | ซอร์สโค้ดฝั่งเซิร์ฟเวอร์ของ pixel-agents (รวม `HermesBridge`) |
| `server/src/hermesBridge.ts` | โค้ดที่รับ `/api/hooks/hermes` แล้วกระจายผ่าน WebSocket |
| `src/` | ซอร์สโค้ดฝั่งเบราว์เซอร์ (หน้า office pixel-art) |
| `docs/ARCHITECTURE.md` | รายละเอียดสถาปัตยกรรมเชิงลึก |
| `docs/PLUGIN-VS-BRIDGE.md` | ทำไมต้องใช้ bridge ไม่ใช่ plugin |
| `docs/TROUBLESHOOTING.md` | ปัญหาที่เจอบ่อย + วิธีแก้ |
| `fixes/` | โน้ตการแก้ปัญหาเฉพาะแพลตฟอร์ม (เช่น Windows build) |

---

## 🔒 ความปลอดภัย

### ค่าเริ่มต้น: `0.0.0.0` = ใครก็เปิดดู office ได้

โดย default office bind กับ `0.0.0.0` หมายความว่า **ใครที่รู้ IP ของ VPS คุณก็เปิดหน้า office ดูได้** แต่:

- ✅ เขาเห็น **แค่กิจกรรม** (ตัวละครขยับ) — ไม่เห็นเนื้อหา conversation ไม่เห็น secret
- 🔐 การ **inject event** (POST ไป `/api/hooks/hermes`) ต้องมี **token** ที่อยู่ใน `server.json` — คนนอกไม่รู้ token ก็ส่งของปลอมเข้าไม่ได้

### ถ้าอยากจำกัด: bind `127.0.0.1` + SSH forward tunnel

แก้ `pixel-office.service`:

```ini
Environment=HOST=127.0.0.1
```

จากนั้นรีสตาร์ท ตอนนี้ office เปิดได้แค่จากในเครื่อง VPS เอง เวลาอยากดูจาก laptop ใช้ **SSH forward tunnel** (ทางเดียวที่ปลอดภัย):

```bash
ssh -L 3100:127.0.0.1:3100 <user>@<YOUR_VPS_IP>
```

แล้วเปิด `http://127.0.0.1:3100` บน laptop — traffic วิ่งเข้ารหัสผ่าน SSH

> ⚠️ **ห้ามใช้ reverse tunnel (`ssh -R`)** กับโปรเจกต์นี้ — มักจะตายด้วย "remote port forwarding failed" และไม่จำเป็นเลย เพราะ office รันบนเครื่องเดียวกับ Hermes อยู่แล้ว ดูรายละเอียดใน `docs/TROUBLESHOOTING.md`

### ไฟล์ที่ห้าม commit เด็ดขาด

เช็ค `.gitignore` ให้แน่ใจว่าครอบคลุม:

- `~/.pixel-agents/server.json` — มี token ของ office
- `.env` — มี secret ของ Hermes
- `id_rsa` / ค่า SSH key

อย่าเผลอ push ขึ้น GitHub

---

## 📚 เอกสารเพิ่มเติม

- 📐 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — สถาปัตยกรรมเชิงลึก (HermesBridge, uuid5, discovery, ทำไมไม่ใช้ Claude path)
- 🧩 [`docs/PLUGIN-VS-BRIDGE.md`](docs/PLUGIN-VS-BRIDGE.md) — ทำไมต้องใช้ bridge ไม่ใช่ plugin pixel_observer
- 🩹 [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) — ปัญหาที่เจอจริง + วิธีแก้

---

## 🙏 Credits & License

โปรเจกต์นี้ยืนพื้นจากงานสองชิ้นของชุมชน:

- **[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)** (MIT) — runtime ตัวจริงของ Hermes agent
- **[pixel-agents-hq/pixel-agents](https://github.com/pixel-agents-hq/pixel-agents)** — เอนจิน office pixel-art
- **[aiunlocked1412/hermes-agent-pixel](https://github.com/aiunlocked1412/hermes-agent-pixel)** — fork ที่เพิ่ม `HermesBridge` และ bridge script เพื่อเชื่อมสองส่วนเข้าด้วยกัน

**License:** MIT — ใช้, ดัดแปลง, แจกจ่ายได้อิสระ

---

> ทำด้วยใจ 💚 เพื่อให้ทุกคนได้เห็น agent ของตัวเอง "มีชีวิต" — ถ้าเจอปัญหา อ่าน `docs/TROUBLESHOOTING.md` ก่อน แล้วค่อยเปิด issue
