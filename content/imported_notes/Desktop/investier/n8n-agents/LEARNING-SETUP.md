# Setup ระบบ AI Learning

## ภาพรวม

```
┌──────────────────────────────────────┐
│ Workflow 1 (รันทุกเช้า 8:00)         │
│                                      │
│ 1. อ่าน "Lessons" จาก Sheets         │
│ 2. ดึงข้อมูลหุ้น                      │
│ 3. Bull → Bear → Judge (รู้บทเรียน)  │
│ 4. บันทึกคำทำนายลง Sheets            │
│ 5. ส่ง Discord                       │
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│ Workflow 2 (รันทุกอาทิตย์ 21:00)    │
│                                      │
│ 1. อ่านคำทำนาย 7 วันที่ผ่านมา       │
│ 2. ดึงราคาปัจจุบันเทียบ              │
│ 3. AI Reflection — เขียนบทเรียน       │
│ 4. บันทึก Lessons ใหม่               │
│ 5. ส่งสรุปประจำสัปดาห์ Discord       │
└──────────────────────────────────────┘
```

## Step 1: สร้าง Google Sheets

1. เปิด https://sheets.google.com
2. สร้างไฟล์ใหม่ ตั้งชื่อ **"AI Stock Memory"**
3. **คัดลอก Spreadsheet ID** จาก URL
   - URL: `https://docs.google.com/spreadsheets/d/【ID อยู่ตรงนี้】/edit`
   - เช่น `1abc...xyz`

## Step 2: สร้าง 2 Tabs

### Tab 1: "Predictions"

แถวแรก (header) ใส่:
```
Date | Symbol | Signal | Confidence | Price | Bull_case | Bear_case | Verdict
```

### Tab 2: "Lessons"

แถวแรก (header) ใส่:
```
Date | Lesson | Performance
```

(double-click ที่แท็บล่างเพื่อเปลี่ยนชื่อ)

## Step 3: เชื่อม Google Sheets ใน n8n

1. กดปุ่ม `+` สร้าง credential ใหม่
2. เลือก **Google Sheets OAuth2 API**
3. กด **Sign in with Google** → login ด้วย michoder26@gmail.com
4. อนุญาต permission

## Step 4: Import Workflow ใหม่

1. Import `workflow.json` (มี learning + logging แล้ว)
2. Import `workflow-reflection.json` (workflow รายสัปดาห์)
3. ในแต่ละ Google Sheets node — ใส่ **Spreadsheet ID** ที่ copy ไว้

## Step 5: ทดสอบ

### ทดสอบ Workflow 1
- Execute manually → ตรวจ Google Sheets ว่ามีแถวใหม่เพิ่ม

### ทดสอบ Workflow 2
- Execute manually (ต้องมีข้อมูลใน Sheets อย่างน้อย 1 วัน)
- ตรวจว่ามี Lesson ใหม่บันทึก

## Maintenance

- Google Sheets ฟรี = 10 ล้าน cell — เพียงพอเป็นปีๆ
- ถ้า lesson เยอะเกินไป → AI จะอ่านแค่ top 10 ล่าสุด
- เปิดดู Sheets เมื่อไหร่ก็ได้ → เห็น track record AI
