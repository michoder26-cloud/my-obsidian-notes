# StimenAufDeutch

เครื่องมือฝึกฟัง-พูดภาษาเยอรมัน ใช้เสียง neural ของ Microsoft (ฟรีผ่าน edge-tts) + แปลไทยอัตโนมัติ

## ติดตั้งครั้งแรก

เปิด PowerShell ในโฟลเดอร์โปรเจค (`C:\Users\TKTF\Desktop\StimenAufDeutch`)

### 1. สร้าง virtual environment

```powershell
python -m venv venv
```

### 2. เปิดใช้ venv

```powershell
.\venv\Scripts\Activate.ps1
```

ถ้าโดน error เรื่อง execution policy ให้รันครั้งเดียวแล้วลองใหม่:
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

ถ้าใช้งาน venv อยู่ จะเห็น `(venv)` นำหน้า prompt

### 3. ติดตั้งไลบรารี

```powershell
pip install -r requirements.txt
```

---

## ใช้งาน

ทุกครั้งที่กลับมาเปิดโปรเจคใหม่ ต้องเปิด venv ก่อน:
```powershell
.\venv\Scripts\Activate.ps1
```

### `try_voices.py` — ลองฟังเสียง 6 แบบ
```powershell
python try_voices.py
```
จะได้ไฟล์ `sample_*.mp3` 6 ไฟล์ เปิดฟังเทียบ เลือกเสียงที่ชอบ

### `speak.py` — พิมพ์ประโยค → ฟังเสียงทันที
```powershell
python speak.py
```

### `shadow.py` — โหมด shadowing สำหรับฝึกพูดตาม
```powershell
python shadow.py
```
- วางข้อความเยอรมันยาวๆ → จบด้วยการกด Enter บนบรรทัดว่าง
- โปรแกรมจะแยกเป็นประโยค อ่านทีละประโยค พร้อมแปลไทย
- คำสั่งระหว่างเล่น:
  - `Enter` = ประโยคถัดไป
  - `r` = ฟังประโยคเดิมซ้ำ
  - `s` = ปรับความเร็ว (เช่น `-20%`, `+0%`, `+10%`)
  - `q` = ออก

---

## ปรับเสียงดีฟอลต์

แก้ตัวแปร `VOICE` ในแต่ละไฟล์ (บรรทัดบนๆ ของไฟล์)

| Voice ID | ลักษณะ |
|---|---|
| `de-DE-KatjaNeural` | หญิง เยอรมนี (นุ่ม ฟังง่าย) — ดีฟอลต์ |
| `de-DE-ConradNeural` | ชาย เยอรมนี (ทุ้ม) |
| `de-DE-KillianNeural` | ชาย เยอรมนี (หนุ่ม) |
| `de-DE-AmalaNeural` | หญิง เยอรมนี (ใส) |
| `de-AT-IngridNeural` | หญิง ออสเตรีย |
| `de-CH-LeniNeural` | หญิง สวิส |

## ปิด venv ตอนเลิกใช้
```powershell
deactivate
```
