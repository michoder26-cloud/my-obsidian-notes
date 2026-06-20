---
name: spawn-agent
description: สร้าง Hermes agent ใหม่ (profile + Telegram bot + gateway)
triggers:
  - "สร้าง agent ใหม่"
  - "เพิ่ม agent"
  - "ตั้ง agent ใหม่"
  - "สร้าง bot ใหม่"
  - "เพิ่ม profile"
---

# SKILL: Spawn New Hermes Agent

## Trigger
> "สร้าง agent ใหม่", "เพิ่ม agent", "ตั้ง agent ใหม่", "สร้าง bot ใหม่", "เพิ่ม profile"

## Prerequisites
- ชื่อ agent ใหม่ (เช่น `analyst`, `writer`, `researcher`)
- Telegram Bot Token (สร้างผ่าน @BotFather)
- Ollama Cloud API Key (ถ้ายังไม่มี)

## Step-by-Step

### Step 1: สร้าง Profile ใหม่
```bash
hermes profile create <ชื่อ-agent>
```

### Step 2: ตั้งค่า Model และ Provider
```bash
hermes --profile <ชื่อ-agent> config set model.provider ollama-cloud
hermes --profile <ชื่อ-agent> config set model.default glm-5.2
hermes --profile <ชื่อ-agent> config set gateway.auth bot
```

### Step 3: เพิ่ม Telegram Bot Token
```bash
# เขียนลง .env ของ profile นั้น
cat > /root/.hermes/profiles/<ชื่อ-agent>/.env << 'EOF'
TELEGRAM_BOT_TOKEN=<token จาก @BotFather>
OLLAMA_API_KEY=<ollama cloud key>
EOF
```

### Step 4: ตั้งค่า Allow User
```bash
hermes --profile <ชื่อ-agent> config set TELEGRAM_ALLOWED_USERS 8675116758
```

### Step 5: รัน Gateway
```bash
hermes --profile <ชื่อ-agent> gateway run &
```

### Step 6: เพิ่มใน Watchdog
```bash
nano /usr/local/bin/gateway-watchdog.sh
# แก้บรรทัด PROFILES=("trader" "coder" "news")
# เป็น PROFILES=("trader" "coder" "news" "<ชื่อ-agent>")
```

### Step 7: Restart Watchdog
```bash
pkill -f gateway-watchdog.sh && sleep 2 && /usr/local/bin/gateway-watchdog.sh &
```

## Verification
```bash
# ตรวจสอบว่า gateway รันอยู่
ps aux | grep "<ชื่อ-agent>" | grep gateway

# ดู log
tail -10 /root/.hermes/profiles/<ชื่อ-agent>/logs/gateway.log

# ทดสอบส่งข้อความไปที่ bot ใน Telegram
```

## Example: สร้าง Analyst Agent
```bash
# 1. สร้าง profile
hermes profile create analyst

# 2. ตั้งค่า
hermes --profile analyst config set model.provider ollama-cloud
hermes --profile analyst config set model.default glm-5.2
hermes --profile analyst config set gateway.auth bot

# 3. เพิ่ม token
cat > /root/.hermes/profiles/analyst/.env << 'EOF'
TELEGRAM_BOT_TOKEN=123456:ABC...
OLLAMA_API_KEY=d27299...
EOF

# 4. รัน
hermes --profile analyst gateway run &

# 5. เพิ่มใน watchdog
# แก้ไข /usr/local/bin/gateway-watchdog.sh
```

## การเลือก Model ตามหน้าที่
| Agent | โมเดลแนะนำ | เหตุผล |
|-------|------------|---------|
| Trader | `glm-5.2` | วิเคราะห์ตลาด, ตัวเลข |
| Coder | `qwen3-coder:480b` | เขียนโค้ดโดยเฉพาะ |
| News | `kimi-k2.6` | Context 1M, สรุปข่าวยาว |
| Analyst | `glm-5.2` | วิเคราะห์ข้อมูล |
| Writer | `glm-5.2` | เขียนเนื้อหา |
| Researcher | `kimi-k2.6` | อ่าน paper ยาว |

## Pitfalls
- **ลืม OLLAMA_API_KEY ใน .env**: ทุก profile ใหม่ต้องมี OLLAMA_API_KEY ใน .env ด้วย ถ้าไม่มี bot จะ Telegram connect สำเร็จแต่ API call ล้มเหลว "Provider authentication failed"
- ลืมสร้าง Telegram Bot ผ่าน @BotFather ก่อน
- API key ไม่ถูกต้อง → 401 error
- Profile name มีช่องว่าง → ใช้ dash แทน (เช่น `data-analyst` ไม่ใช่ `data analyst`)
- .env ต้องอยู่ใน `/root/.hermes/profiles/<ชื่อ>/.env` ไม่ใช่ที่อื่น
- Watchdog ต้องเพิ่ม profile ใหม่ใน PROFILES array ด้วย ไม่งั้นจะไม่ auto-restart

## Telegram Bot Creation
1. ไปที่ @BotFather ใน Telegram
2. พิมพ์ `/newbot`
3. ตั้งชื่อ bot (เช่น "Data Analyst Bot")
4. ตั้ง username (ต้องลงท้ายด้วย `bot` เช่น `DataAnalyst123_bot`)
5. คัดลอก token ที่ได้มาใช้ใน step 3
