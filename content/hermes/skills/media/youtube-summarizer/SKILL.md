---
name: youtube-summarizer
description: ใช้เมื่อต้องการดู YouTube video แล้วให้ AI สรุป — รองรับภาษาไทย, อังกฤษ, เยอรมัน, บันทึกเป็น .md หรือ skill ใหม่
version: 1.0.0
author: User
license: MIT
metadata:
  hermes:
    tags: [youtube, summary, learning, thai, german, english]
    related_skills: [youtube-content, claw-trade-mt5-linux]
---

# YouTube Video Summarizer → Skill Creator

## ภาพรวม
Skill นี้ใช้สำหรับ:
1. รับ URL YouTube จากคุณ
2. ดึง subtitle/transcript อัตโนมัติ (ไทย, อังกฤษ, เยอรมัน)
3. สรุปว่า **วิดีโอสอนอะไร**
4. **สร้างเป็น skill ใหม่** ให้คุณเรียกใช้ได้เลย!

**ไม่ต้อง save .md แยก** — ผลลัพธ์คือ skill ที่ใช้งานได้จริง

## เมื่อไรใช้
- คุณแปะลิงก์ YouTube แล้วพูดว่า "ดูให้หน่อย" หรือ "สรุปให้"
- คุณเห็นวิดีโอสอนเทรด / coding / ตั้งค่าระบบ แล้วอยากให้ผมจำ workflow นั้นไว้
- ต้องการแปลงเนื้อหาจาก YouTube → skill ที่เรียกใช้ได้ทันที

## ภาษา — รองรับอัตโนมัติ

| ภาษา | รหัส | ค้นหา |
|------|------|-------|
| ภาษาไทย | `th` | ✅ อัตโนมัติ |
| English | `en` | ✅ อัตโนมัติ |
| Deutsch | `de` | ✅ อัตโนมัติ |
| ภาษาอื่น | auto | fallback |

## ขั้นตอนการทำงาน

### 1. ดึง Transcript
ใช้ script `fetch_transcript.py` — รองรับ URL ทุกรูปแบบ:
```bash
python3 /root/.hermes/skills/media/youtube-content/scripts/fetch_transcript.py \
  "URL" \
  --language th,en,de \
  --timestamps \
  --text-only
```

### 2. สรุปเนื้อหา (AI)
ผมจะสรุปให้คุณเป็นภาษาไทย:
- **หัวข้อ:** วิดีโอนี้สอนเกี่ยวกับอะไร
- **เนื้อหาหลัก:** สรุปทีละประเด็น
- **สิ่งที่เรียนรู้ได้:** takeaways สำหรับระบบ/การเทรดของคุณ

### 3. ถามคุณก่อนสร้าง skill
ผมจะถามเสมอว่า "สร้าง skill จากเนื้อหานี้ไหม?" — แล้วรอคำตอบคุณ

### 4. สร้าง skill (เมื่อคุณตอบตกลง)
ใช้ `skill_manage(action='create')` สร้าง skill ใหม่:
- ชื่อ skill = สั้น กระชับ ตรงกับเนื้อหา
- category = media หรือ software-development (ตามเนื้อหา)
- เนื้อหา = ขั้นตอนจากวิดีโอที่ปรับเป็น workflow ที่ใช้ได้จริง
- **skill นี้จะอยู่ใน skills list ของผมทันทีใน session ใหม่**

## Pitfalls
1. **วิดีโอไม่มี subtitle** → ไม่สามารถดึง transcript ได้ — บอกคุณทันที
2. **วิดีโอส่วนตัว/ถูกลบ** — บอก error
3. **Transcript ยาวเกินไป** — จะแบ่ง chunk แล้วสรุปทีละส่วน
4. **Script ใช้ youtube-transcript-api** — ต้องติดตั้งก่อน: `pip install youtube-transcript-api`
5. **Skill ที่สร้างใหม่จะใช้ได้ใน session ถัดไป** — session ปัจจุบันจะไม่เห็นจนกว่าจะเปิดใหม่
6. **ถามคุณทุกครั้งก่อนสร้าง** — ไม่สร้าง skill โดยไม่ถาม

## ตัวอย่างคำสั่ง
```
"ดูวิดีโอนี้ให้หน่อย https://youtube.com/watch?v=xxx"
"สรุปคลิปนี้ https://youtu.be/xxx และสร้าง skill ให้ด้วย"
"https://youtube.com/watch?v=xxx ช่วยดูให้หน่อย"
```

## Verification
- [ ] script ดึง transcript ได้
- [ ] สรุปครบถ้วน (หัวข้อ + เนื้อหา + takeaways)
- [ ] ถามคุณก่อนสร้าง skill
- [ ] skill ถูกสร้างด้วย skill_manage (action='create')