# Telegram Group Chat Setup

## Overview
Enable all Hermes agent bots to respond in a shared Telegram group. Bots respond to @mentions by default, preventing noise.

## Prerequisites
- All agent gateways running (each bot must be online)
- A Telegram group created by the user
- Bot usernames known for each agent

## Steps

### 1. Disable Privacy Mode (critical)
Bot จะไม่เห็น group messages ถ้า privacy mode เปิดอยู่:
1. ไปที่ @BotFather → /mybots
2. เลือกแต่ละ bot
3. Bot Settings → Group Privacy → Turn **OFF**
4. เอา bot ออกจาก group → ใส่กลับเข้าไปใหม่

ทำให้ **ทุก bot** ที่จะใช้ใน group (default, trader, coder, news, system)

### 2. Add bots to group
User ทำเองใน Telegram:
- `@hermesAiVPS1234_bot` (default)
- `@GoldTraderLLM123_bot` (trader)
- `@CoderLLM123_bot` (coder)
- `@NewsCollectorLLM123_bot` (news)
- `@SystemLLM122_bot` (system)

### 3. Get the group chat ID
**วิธี A (แนะนำ):** ดูจาก gateway log ตอนที่ bot ถูก add เข้า group:
```bash
# ดู log ที่เก่าที่สุด (ก่อนที่ gateway จะ consume updates)
journalctl --user -u hermes-gateway -o cat --since "30 min ago" | grep -i "new_chat_participant\|group"
# หรือดูจาก hermes state
python3 -c "
import json
with open('/root/.hermes/gateway_state.json') as f:
    d=json.load(f)
    print(json.dumps(d, indent=2))
"
```

**วิธี B:** Stop gateway ชั่วคราว → poll API → restart:
```bash
pkill -f "hermes.*gateway run"          # stop ALL
sleep 1
token=$(python3 -c "
with open('/root/.hermes/.env','r') as f:
    for l in f:
        if l.startswith('TELEGRAM_BOT_TOKEN='):
            print(l.strip().split('=',1)[1])
            break
")
curl -s "https://api.telegram.org/bot${token}/getUpdates" | python3 -c "
import json, sys
d=json.load(sys.stdin)
for u in d.get('result',[]):
    c=u.get('message',{}).get('chat',{})
    if c.get('type') in ('group','supergroup'):
        print(f'Title={c[\"title\"]}, ID={c[\"id\"]}')
"
# แล้ว restart gateway ทุกตัวใหม่
```

**วิธี C:** ถ้า default bot ตอบใน group ได้แล้ว → group ID อยู่ใน session:
- เช็คจาก `~/.hermes/state/` JSON ไฟล์
- หรือใช้ `session_search` หาคำที่ default bot ตอบใน group

Group chat ID format: `-100` + 9+ digits (supergroup) เช่น `-1002585512037`

### 4. Configure allowlist ให้ทุก agent
เพิ่ม group ID + user ID ลง `.env` ทุก profile:
```bash
python3 << 'EOF'
import os
GROUP_ID = "-1002585512037"      # เปลี่ยนเป็น ID ของคุณ
USER_ID  = "8675116758"           # เปลี่ยนเป็น ID ของคุณ
ALLOWED  = f"TELEGRAM_ALLOWED_USERS={USER_ID},{GROUP_ID}\n"
GROUP_LINE = f"TELEGRAM_GROUP_CHAT_ID={GROUP_ID}\n"

for p in ['trader', 'coder', 'news', 'system', 'default']:
    path = f"/root/.hermes/profiles/{p}/.env"
    with open(path, 'a') as f:
        f.write(ALLOWED)
        f.write(GROUP_LINE)
    print(f"✅ Updated {p}")
EOF
```

> ใช้ Python เพราะ shell echo จะ censor Telegram tokens (ระบบ replace colon+value เป็น `***`)

### 5. Restart ทุก gateway
```bash
# 1. Kill ทุกตัว (ยกเว้นตัวที่ตัวเองรันผ่าน terminal — อย่า kill ตัวเอง!)
for p in trader coder news system; do
    pkill -f "hermes.*profile $p.*gateway"
    sleep 1
    rm -f ~/.hermes/profiles/$p/gateway.lock ~/.hermes/profiles/$p/gateway.pid
done

# 2. Start ทุกตัวพร้อมกัน (ใช้ terminal(background=true))
export PATH="/usr/local/lib/hermes-agent/venv/bin:$HOME/.local/bin:$PATH"
for p in trader coder news system; do
    hermes --profile $p gateway run &   # ใช้ terminal(background=true) แทนถ้าใช้ Hermes tool
    sleep 0.5
done
```

### 6. Test
ใน Telegram group พิมพ์:
- `@GoldTraderLLM123_bot วิเคราะห์ทองหน่อย`
- `@CoderLLM123_bot เขียน Python hello world`

ถ้าไม่ตอบ → เช็ค journalctl ว่า gateway connect สำเร็จไหม:
```bash
journalctl --user -u hermes-gateway -n 20
```

## How group chat works in Hermes
- Bots receive group messages via Telegram long-polling
- Bot ตอบเมื่อถูก @mention เท่านั้น (default behavior)
- แต่ละ bot มี session แยกตาม chat ID (group ≠ DM)
- `TELEGRAM_ALLOWED_USERS` ต้องมี group ID ถึงจะ respond ใน group

## Pitfalls
- **Privacy mode ON = bots blind:** ถ้าไม่ disable privacy ใน @BotFather → bot ไม่เห็น messages → ไม่ respond
- **getUpdates returns empty:** Gateway กิน updates แล้ว → ต้อง read logs แทน หรือ stop gateway ชั่วคราว
- **Token censorship breaks .env:** Shell `echo 'TELEGRAM_BOT_TOKEN=123:abc'` จะถูก censor เป็น `123:***` → write ผิด → gateway start แต่ auth fail → เหมือน "บอทไม่รัน" แต่จริงๆ รันอยู่แค่ token ผิด
- **kill -9 all เกิด loop:** อย่า `pkill -9` ทุก gateway พร้อมกัน → ถ้า gateway-monitor cron วิ่งอยู่ มันจะ restart ทันที กลายเป็น race condition
- **OOM after gateway swarm:** ถ้า start 5 gateway พร้อมกัน → RAM spike → OOM → ดับหมด → ต้อง stagger start 0.5s ต่อตัว