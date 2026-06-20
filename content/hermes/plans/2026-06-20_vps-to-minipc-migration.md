# ย้ายระบบจาก VPS → Minisforum UM790 Pro Mini PC

> **For Hermes:** ปฏิบัติตามแผนทีละขั้นตอน ห้ามข้ามขั้น

**Goal:** ย้าย Hermes Agent (4 profiles) + ClawTrade Bot + Pixel Agent Office จาก VPS ปัจจุบันไปยัง Minisforum UM790 Pro (Debian 12 GUI) โดยระบบทำงานเหมือนเดิมทุกอย่าง

**เครื่องเป้าหมาย:** Minisforum UM790 Pro · Ryzen 9 7940HS · 64GB RAM · 269GB SSD · Debian 12 (GUI ติดตั้งแล้ว)

**VPS ปัจจุบัน:** Linux 6.8.0 · 12GB RAM · 193GB disk · Hermes v0.16.0 · Docker + MT5/Wine

---

## ข้อมูลที่ต้องย้าย

| รายการ | ขนาด | Path |
|---|---|---|
| Hermes ทั้งหมด | 1.8 GB | `/root/.hermes/` |
| ClawTrade | 756 MB | `/root/Claw_Trade/` |
| Pixel Agent Office | 540 KB | `/root/pixel-agent-office/` |
| Docker volume: claw-trade-config | 2.9 GB | MT5 config |
| Docker volume: mt5_config | 3.7 GB | MT5 terminal |
| Scripts | ~5 KB | `/usr/local/bin/gateway-*.sh`, `hermes-services.sh` |
| systemd service | 1 file | `/etc/systemd/system/hermes-gateway-watchdog.service` |
| crontab | 3 entries | `crontab -l` |
| Hermes cron jobs | 2 jobs | trader (ClawTrade watchdog 15min) + system (report 5h) |
| ClawTrade logs | ~small | `/tmp/claw_live*.log` |
| **รวม** | **~9.2 GB** | (ไม่รวม Docker images ที่ดึงใหม่) |

---

## ขั้นตอนทั้งหมด 7 ขั้น

```
ขั้น 1: เตรียม Mini PC (ติดตั้งซอฟต์แวร์)
ขั้น 2: สำรองข้อมูลจาก VPS
ขั้น 3: โอนข้อมูล VPS → Mini PC
ขั้น 4: คืนค่าข้อมูลบน Mini PC
ขั้น 5: ติดตั้ง Docker + MT5 Container
ขั้น 6: อัปเดต Config (IP, Webhook, Firewall)
ขั้น 7: ทดสอบระบบ + ปิด VPS
```

---

## ขั้น 1: เตรียม Mini PC (ติดตั้งซอฟต์แวร์)

**วัตถุประสงค์:** ติดตั้งทุกอย่างที่จำเป็นบน Mini PC

### 1.1 เปิด Terminal บน Mini PC (Applications → Terminal)

### 1.2 อัปเดตระบบ

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.3 ติดตั้งซอฟต์แวร์พื้นฐาน

```bash
sudo apt install -y \
  openssh-server \
  curl \
  git \
  python3 \
  python3-venv \
  python3-pip \
  build-essential \
  jq \
  tar \
  rsync \
  ufw \
  fail2ban
```

### 1.4 เปิด SSH (ถ้าต้องการเข้าจากเครื่องอื่น)

```bash
sudo systemctl enable --now ssh
```

### 1.5 ตั้งค่า Firewall

```bash
sudo ufw allow 22/tcp       # SSH
sudo ufw allow 9119/tcp     # Hermes Dashboard
sudo ufw allow 9120/tcp     # Pixel Agent Office
sudo ufw allow 3000/tcp     # MT5 VNC (web)
sudo ufw allow 8001/tcp     # MT5 RPyC
sudo ufw enable
```

### 1.6 ติดตั้ง Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# logout แล้ว login ใหม่ หรือ: newgrp docker
```

### 1.7 ติดตั้ง Hermes Agent

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install | bash
```

ตรวจสอบ:

```bash
hermes --version
# ควรเห็น: Hermes Agent v0.16.0 หรือใหม่กว่า
```

### 1.8 ติดตั้ง Python dependencies สำหรับ ClawTrade

```bash
cd /root/Claw_Trade
python3 -m venv venv
source venv/bin/activate
pip install anthropic==0.28.0 python-dotenv==1.0.0 yfinance==0.2.32 \
  pandas==2.1.3 numpy==1.24.3 requests==2.31.0 pydantic==2.5.0 \
  httpx==0.25.2 MetaTrader5 pandas-ta
```

**ตรวจสอบ:**

```bash
docker --version          # Docker 24+
hermes --version          # Hermes Agent v0.16.0+
python3 --version         # Python 3.11+
ssh localhost             # SSH ทำงาน
```

---

## ขั้น 2: สำรองข้อมูลจาก VPS

**วัตถุประสงค์:** สำรองข้อมูลทั้งหมดจาก VPS ปัจจุบันเป็นไฟล์ tar

### 2.1 สำรองไฟล์ระบบ

```bash
# รันบน VPS
tar czf /tmp/vps-files-backup.tar.gz \
  -C / \
  root/.hermes/ \
  root/Claw_Trade/ \
  root/pixel-agent-office/ \
  usr/local/bin/gateway-watchdog.sh \
  usr/local/bin/gateway-services.sh \
  usr/local/bin/hermes-services.sh \
  usr/local/bin/gateway-monitor.sh \
  etc/systemd/system/hermes-gateway-watchdog.service
```

### 2.2 สำรอง crontab

```bash
crontab -l > /tmp/vps-crontab.txt
```

### 2.3 สำรอง Docker volumes

```bash
# claw-trade-config
docker run --rm -v claw-trade-config:/data -v /tmp:/backup \
  alpine tar czf /backup/claw-trade-config.tar.gz -C /data .

# mt5_config
docker run --rm -v mt5_config:/data -v /tmp:/backup \
  alpine tar czf /backup/mt5_config.tar.gz -C /data .
```

### 2.4 ตรวจสอบไฟล์สำรอง

```bash
ls -lh /tmp/vps-files-backup.tar.gz \
       /tmp/vps-crontab.txt \
       /tmp/claw-trade-config.tar.gz \
       /tmp/mt5_config.tar.gz
```

**คาดว่าขนาดไฟล์:**
- vps-files-backup.tar.gz: ~1.5 GB (Hermes + ClawTrade + Pixel)
- claw-trade-config.tar.gz: ~1 GB
- mt5_config.tar.gz: ~1.5 GB

---

## ขั้น 3: โอนข้อมูล VPS → Mini PC

**วัตถุประสงค์:** ส่งไฟล์สำรองจาก VPS ไปยัง Mini PC

### 3.1 หา IP ของ Mini PC

```bash
# รันบน Mini PC
ip addr show | grep 'inet ' | grep -v 127.0.0.1
# จด IP ไว้ (เช่น 192.168.1.100)
```

### 3.2 โอนไฟล์ผ่าน SCP (รันบน VPS)

```bash
# แทน MINI_PC_IP ด้วย IP จริงของ Mini PC
scp /tmp/vps-files-backup.tar.gz root@MINI_PC_IP:/root/
scp /tmp/vps-crontab.txt root@MINI_PC_IP:/root/
scp /tmp/claw-trade-config.tar.gz root@MINI_PC_IP:/root/
scp /tmp/mt5_config.tar.gz root@MINI_PC_IP:/root/
```

### 3.3 หรือโอนผ่าน rsync (เร็วกว่า และต่อได้ถ้าขาด)

```bash
rsync -avz --progress /tmp/vps-files-backup.tar.gz root@MINI_PC_IP:/root/
rsync -avz --progress /tmp/claw-trade-config.tar.gz root@MINI_PC_IP:/root/
rsync -avz --progress /tmp/mt5_config.tar.gz root@MINI_PC_IP:/root/
```

### 3.4 ตรวจสอบว่าไฟล์มาครบ

```bash
# รันบน Mini PC
ls -lh /root/vps-files-backup.tar.gz \
       /root/vps-crontab.txt \
       /root/claw-trade-config.tar.gz \
       /root/mt5_config.tar.gz
```

---

## ขั้น 4: คืนค่าข้อมูลบน Mini PC

**วัตถุประสงค์:** แตกไฟล์สำรองและวางไฟล์ที่ถูกที่

### 4.1 คืนค่าไฟล์ระบบ

```bash
# รันบน Mini PC
cd /
sudo tar xzf /root/vps-files-backup.tar.gz -C /

# ตรวจสอบ
ls -la /root/.hermes/
ls -la /root/Claw_Trade/
ls -la /root/pixel-agent-office/
ls -la /usr/local/bin/gateway-watchdog.sh
ls -la /usr/local/bin/hermes-services.sh
ls -la /etc/systemd/system/hermes-gateway-watchdog.service
```

### 4.2 ตั้ง permission ให้ scripts

```bash
sudo chmod +x /usr/local/bin/gateway-watchdog.sh
sudo chmod +x /usr/local/bin/hermes-services.sh
sudo chmod +x /usr/local/bin/gateway-monitor.sh
```

### 4.3 คืนค่า crontab

```bash
crontab /root/vps-crontab.txt
crontab -l  # ตรวจสอบ
```

### 4.4 เปิดใช้ systemd service

```bash
sudo systemctl daemon-reload
sudo systemctl enable hermes-gateway-watchdog.service
```

### 4.5 สร้างโฟลเดอร์ log

```bash
sudo mkdir -p /var/log/gateway-watchdog
```

---

## ขั้น 5: ติดตั้ง Docker + MT5 Container

**วัตถุประสงค์:** สร้าง MT5 container บน Mini PC ให้เหมือนเดิม

### 5.1 ดึง Docker image

```bash
docker pull gmag11/metatrader5_vnc
```

### 5.2 คืนค่า Docker volumes

```bash
# สร้าง volumes
docker volume create claw-trade-config
docker volume create mt5_config

# คืนค่า claw-trade-config
docker run --rm -v claw-trade-config:/data -v /root:/backup \
  alpine tar xzf /backup/claw-trade-config.tar.gz -C /data

# คืนค่า mt5_config
docker run --rm -v mt5_config:/data -v /root:/backup \
  alpine tar xzf /backup/mt5_config.tar.gz -C /data
```

### 5.3 สร้าง container (เหมือนเดิมทุกอย่าง)

```bash
docker run -d \
  --name claw-trade-mt5 \
  --restart unless-stopped \
  -p 3000:3000 \
  -p 8001:8001 \
  -v claw-trade-config:/config \
  gmag11/metatrader5_vnc
```

### 5.4 รอ MT5 boot และตรวจสอบ

```bash
sleep 30
docker ps | grep claw
# ควรเห็น: claw-trade-mt5 ... Up

# ตรวจ RPyC
curl -s http://localhost:8001 || echo "RPyC not ready yet (อาจต้องรอ 1-2 นาที)"
```

### 5.5 ตรวจสอบ MT5 login (ผ่าน VNC web)

เปิดเบราว์เซอร์: `http://localhost:3000`
- ถ้าเห็นหน้า VNC → MT5 ทำงาน
- ถ้า MT5 ยังไม่ login → ต้อง login ใหม่ด้วยมือ (อาจต้องใส่รหัสใหม่)

---

## ขั้น 6: อัปเดต Config (IP, Webhook, Firewall)

**วัตถุประสงค์:** แก้ไข config ที่อ้างถึง IP ของ VPS เดิม

### 6.1 หา IP เดิมของ VPS

```bash
# รันบน VPS
curl -s ifconfig.me
# จด IP ไว้ (เช่น 13.140.183.183)
```

### 6.2 หา IP ใหม่ของ Mini PC

```bash
# รันบน Mini PC
# ถ้ามี public IP (ถ้าเสียบสาย LAN ตรง router):
curl -s ifconfig.me

# ถ้าเป็น private IP (อยู่หลัง router):
ip addr show | grep 'inet ' | grep -v 127.0.0.1
```

### 6.3 อัปเดต IP ใน Hermes config

```bash
# แทน OLD_IP ด้วย IP เดิม, NEW_IP ด้วย IP ใหม่
sudo sed -i 's/OLD_IP/NEW_IP/g' /root/.hermes/config.yaml
sudo sed -i 's/OLD_IP/NEW_IP/g' /root/.hermes/profiles/*/config.yaml

# ตรวจสอบ
grep -r "OLD_IP" /root/.hermes/  # ไม่ควรเจอ
```

### 6.4 อัปเดต Pixel Agent Office

```bash
# ถ้ามีการอ้างอิง IP ใน server.py
grep "OLD_IP" /root/pixel-agent-office/server.py
sudo sed -i 's/OLD_IP/NEW_IP/g' /root/pixel-agent-office/server.py
```

### 6.5 อัปเดต Telegram Webhook (ถ้าใช้ webhook mode)

```bash
# ตรวจสอบว่า Hermes ใช้ webhook หรือ polling
grep -r "webhook" /root/.hermes/config.yaml
grep -r "webhook" /root/.hermes/profiles/*/config.yaml

# ถ้าใช้ webhook ต้องอัปเดต:
# ดูว่าแต่ละ profile มี bot token อะไร
for p in trader coder news system; do
  echo "--- $p ---"
  grep -A2 "telegram" /root/.hermes/profiles/$p/config.yaml | grep -i "token\|webhook"
done

# อัปเดต webhook ผ่าน Telegram API:
# curl -s "https://api.telegram.org/bot<TOKEN>/setWebhook?url=https://NEW_IP:443/bot<TOKEN>"
```

### 6.6 ตั้งค่า Port Forwarding (ถ้า Mini PC อยู่หลัง router)

เข้า router admin → Port Forwarding:
- Port 9119 → Mini PC IP (Dashboard)
- Port 9120 → Mini PC IP (Pixel Agent)
- Port 3000 → Mini PC IP (MT5 VNC) [ถ้าต้องเข้าจากนอกบ้าน]
- Port 8001 → Mini PC IP (RPyC) [ถ้าต้อง]

หรือใช้ **Tailscale / Cloudflare Tunnel** (ง่ายกว่า ไม่ต้อง port forward):

```bash
# ติดตั้ง Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
# จะได้ IP คงที่ เข้าถึงได้จากทุกที่
```

### 6.7 อัปเดต Dashboard URL ใน report cron

```bash
# ถ้า cron prompt มีการอ้าง URL ของ dashboard
# ต้องอัปเดต IP ใน prompt ของ cron jobs ด้วย
# แก้ผ่าน: hermes --profile system cron list
# และอัปเดต prompt ถ้าจำเป็น
```

---

## ขั้น 7: ทดสอบระบบ + ปิด VPS

**วัตถุประสงค์:** ทดสอบว่าทุกอย่างทำงานบน Mini PC ก่อนปิด VPS

### 7.1 เริ่ม Hermes Gateways

```bash
# รัน hermes-services.sh (เริ่ม gateway + dashboard)
bash /usr/local/bin/hermes-services.sh &

# รอ 10 วินาที
sleep 10

# ตรวจสอบ gateways ทั้ง 4
for p in trader coder news system; do
  echo -n "$p: "
  python3 -c "import json; d=json.load(open(f'/root/.hermes/profiles/$p/gateway_state.json')); print(d['gateway_state'], d['platforms']['telegram']['state'])" 2>/dev/null
done
# คาดว่า: ทั้ง 4 ขึ้น "running connected"
```

### 7.2 เริ่ม gateway-watchdog

```bash
sudo systemctl start hermes-gateway-watchdog
sudo systemctl status hermes-gateway-watchdog
```

### 7.3 ทดสอบ Dashboard

```bash
curl -s http://localhost:9119/ | head -1
# คาดว่า: มีการตอบกลับ (302 หรือ 200)
```

### 7.4 ทดสอบ ClawTrade

```bash
# Container
docker ps | grep claw
# คาดว่า: claw-trade-mt5 Up

# เริ่ม bot
cd /root/Claw_Trade
source venv/bin/activate
python3 main.py live --confirm --symbol XAUUSDc --interval 5 > /tmp/claw_live7.log 2>&1 &

# รอ 30 วินาที แล้วตรวจสอบ
sleep 30
ps aux | grep 'main.py live' | grep -v grep
tail -20 /tmp/claw_live7.log
# คาดว่า: บอททำงาน เชื่อมต่อ MT5 ผ่าน RPyC ได้
```

### 7.5 ทดสอบ Pixel Agent Office

```bash
cd /root/pixel-agent-office
python3 server.py &
sleep 3
curl -s http://localhost:9120/api/status | head -5
```

### 7.6 ทดสอบ Telegram

ส่งข้อความไปยัง Telegram bot ของแต่ละ profile:
- ส่ง "สวัสดี" ไปยัง bot ของ trader → ควรตอบ
- ส่ง "สวัสดี" ไปยัง bot ของ system → ควรตอบ
- ทำซ้ำกับ coder, news

### 7.7 ทดสอบ Cron Jobs

```bash
# ทดสอบ ClawTrade watchdog (profile: trader)
hermes --profile trader cron list

# ทดสอบ System Agent report (profile: system)
hermes --profile system cron list

# รอจนถึงรอบ cron ถัดไป หรือ trigger ด้วยมือ:
hermes --profile trader cron run bd695ea2081d
hermes --profile system cron run c710734269a2
```

### 7.8 ระบบ Checklist ก่อนปิด VPS

```bash
# รันบน Mini PC — ทุกข้อต้องผ่าน
echo "=== Final Checklist ==="

echo "1. Gateways:"
for p in trader coder news system; do
  state=$(python3 -c "import json; d=json.load(open(f'/root/.hermes/profiles/$p/gateway_state.json')); print(d['gateway_state'])" 2>/dev/null)
  echo "   $p: $state"
done

echo "2. Dashboard:"
curl -s -o /dev/null -w "   HTTP %{http_code}" http://localhost:9119/
echo ""

echo "3. Docker:"
docker ps --format "   {{.Names}}: {{.Status}}" | grep claw

echo "4. ClawTrade Bot:"
ps aux | grep 'main.py live' | grep -v grep | awk '{print "   PID:", $2}'

echo "5. Watchdog:"
sudo systemctl is-active hermes-gateway-watchdog

echo "6. Cron:"
crontab -l | grep -v '^#' | grep -v '^$' | head -5

echo "7. Pixel Agent:"
curl -s -o /dev/null -w "   HTTP %{http_code}" http://localhost:9120/
echo ""

echo "8. Telegram:"
for p in trader coder news system; do
  state=$(python3 -c "import json; d=json.load(open(f'/root/.hermes/profiles/$p/gateway_state.json')); print(d['platforms']['telegram']['state'])" 2>/dev/null)
  echo "   $p telegram: $state"
done
```

### 7.9 ปิด VPS

เมื่อทุกข้อใน checklist ผ่าน:

```bash
# รอ 24 ชั่วโมง เพื่อดูว่าระบบเสถียร
# ถ้าทุกอย่างปกติ → ปิด VPS ได้
```

---

## ⚠️ ข้อควรระวัง

| ปัญหา | วิธีแก้ |
|---|---|
| **MT5 ล็อกอินใหม่** — Wine prefix อาจไม่เข้ากับเครื่องใหม่ | เข้า VNC ที่ `localhost:3000` แล้ว login MT5 ใหม่ด้วยมือ |
| **Telegram webhook** ชี้ไป IP เดิม | อัปเดต webhook หรือเปลี่ยนเป็น polling mode |
| **IP เปลี่ยน** — ถ้า Mini PC อยู่หลัง router จะได้ private IP | ใช้ Tailscale หรือ Cloudflare Tunnel เพื่อเข้าถึงจากนอกบ้าน |
| **Docker image ใหญ่ ~7GB** — ดึงนาน | ดึงล่วงหน้าได้ตั้งแต่ขั้น 1 |
| **ClawTrade venv** — Python packages อาจต่างกัน | สร้าง venv ใหม่บน Mini PC (ขั้น 1.8) |
| **SSH key** — ถ้า VPS ใช้ key auth | คัดลอก `~/.ssh/` ไปด้วย |
| **Hermes session DB** — ประวัติการสนทนาเก่า | ย้ายไปด้วยใน `/root/.hermes/` แต่ path อาจต้องปรับ |

---

## ลำดับความสำคัญ (ทำตามลำดับ)

```
ขั้น 1  → ติดตั้งซอฟต์แวร์บน Mini PC         (30 นาที)
ขั้น 2  → สำรองข้อมูลจาก VPS                (15 นาที)
ขั้น 3  → โอนข้อมำล VPS → Mini PC           (20 นาที ขึ้นกว่าความเร็วเน็ต)
ขั้น 4  → คืนค่าข้อมูลบน Mini PC            (10 นาที)
ขั้น 5  → ติดตั้ง Docker + MT5              (30 นาที + รอ MT5 login)
ขั้น 6  → อัปเดต Config + Webhook          (15 นาที)
ขั้น 7  → ทดสอบ + ปิด VPS                  (24 ชม. สังเกต)
```

**รวมเวลาติดตั้ง: ~2 ชั่วโมง** (ไม่รวมรอ 24 ชม. สังเกต)