# Watchdog Config — เพิ่ม Agent ใหม่

## ตำแหน่งไฟล์
```
/usr/local/bin/gateway-watchdog.sh
```

## วิธีเพิ่ม Agent ใหม่ใน Watchdog

### 1. แก้ไข PROFILES array
```bash
nano /usr/local/bin/gateway-watchdog.sh
```
เปลี่ยนบรรทัด:
```bash
PROFILES=("trader" "coder" "news")
```
เป็น:
```bash
PROFILES=("trader" "coder" "news" "<ชื่อ-agent-ใหม่>")
```

### 2. Restart Watchdog
```bash
pkill -f gateway-watchdog.sh && sleep 2 && /usr/local/bin/gateway-watchdog.sh &
```

### 3. ตรวจสอบ
```bash
ps aux | grep gateway-watchdog | grep -v grep
```

## ⚠️ สำคัญ: Watchdog ต้องเงียบ (SILENT)
- ห้ามส่ง Telegram notification ถึง user
- ถ้า agent ล่ม → restart เงียบๆ ไม่ต้องบอก
- User บอกว่า "มันน่าลำคาญ" ถ้าได้ notification
- ดู pattern ที่ถูกต้องใน `/usr/local/bin/gateway-watchdog.sh` ที่ใช้งานจริง

## หมายเหตุ
- Watchdog รันทุก 30 วินาที (config: CHECK_INTERVAL)
- ถ้า agent ล่ม จะ restart อัตโนมัติ
- ไม่ส่ง notification ใน Telegram (silent mode)
