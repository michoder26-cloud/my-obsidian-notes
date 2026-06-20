# 🤖 Multi-Agent System - Agents คุยกันเอง

ระบบนี้ให้ Claude agents หลายตัวคุยกันเอง แล้วทำงานให้คุณ

---

## **📋 ระบบทำงานยังไง**

```
คุณ ให้งาน
  ↓
👔 MANAGER (วางแผน) → "ต้องให้ Analyzer กับ Developer ช่วย"
  ↓
  ├─→ 📊 ANALYZER → "วิเคราะห์ว่าต้องทำอะไร"
  │   (บอกผล → Manager)
  │
  └─→ 💻 DEVELOPER → "เขียนโค้ด/แก้ปัญหา"
      (บอกผล → Manager)
  ↓
👔 MANAGER → "สรุปผลลัพธ์ให้คุณ"
```

---

## **🚀 วิธีใช้**

### 1️⃣ **ติดตั้ง dependencies**
```bash
pip install -r requirements.txt
```

### 2️⃣ **ตั้ง API Key**
```bash
# Windows PowerShell
$env:ANTHROPIC_API_KEY = "sk-ant-xxxxx"

# หรือใน .env file
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

### 3️⃣ **รัน script**
```bash
python main_orchestrator.py
```

### 4️⃣ **เปลี่ยนงาน**
เปิด `main_orchestrator.py` แล้วแก้ไข:
```python
task = "งานใหม่ที่คุณต้องการ"
run_multi_agent_system(task)
```

---

## **✨ ตัวอย่างงาน**

```python
# ตัวอย่าง 1: สร้างเว็บไซต์
task = "สร้าง React website สำหรับ e-commerce"

# ตัวอย่าง 2: วิเคราะห์ข้อมูล
task = "วิเคราะห์ sales data และ สร้าง report"

# ตัวอย่าง 3: ลบบั๊ก
task = "มี error: 'TypeError: cannot read property x of undefined' แก้ให้"
```

---

## **🔄 Agents คุยกันยังไง**

1. **Manager** รับงานจากคุณ
2. **Manager** วางแผน → บอก Analyzer ว่าต้องวิเคราะห์อะไร
3. **Analyzer** ตอบกลับ → ส่งผลวิเคราะห์
4. **Manager** บอก Developer → "จากผลวิเคราะห์ ให้ทำอย่างนี้"
5. **Developer** ตอบกลับ → ส่งโค้ด/แก้ไข
6. **Manager** สรุปผล → ส่งให้คุณ

---

## **🎯 ข้อดี**

✅ Agents ทำงานตามลำดับ  
✅ Agents คุยกันเอง ไม่ต้องคุณจัดการ  
✅ ได้ผลลัพธ์ที่ดีกว่า (หลาย perspective)  
✅ เหมือนมีทีม AI ทำงานให้คุณ  

---

## **💡 ปรับแต่งเพิ่มเติม**

### เพิ่ม Agent ใหม่
```python
def tester_agent(task: str):
    """Sub Agent 3 - ทดสอบโค้ด"""
    conversations["tester"].append({
        "role": "user",
        "content": f"ทดสอบโค้ดนี้: {task}"
    })
    # ... ส่วนที่เหลือเหมือน
```

### เปลี่ยน Model
```python
response = client.messages.create(
    model="claude-opus-4-1",  # เปลี่ยนเป็น model อื่น
    max_tokens=1000,
    messages=conversations["manager"]
)
```

---

## **📞 Support**

ถ้ามี error:
1. ตรวจ API Key
2. ตรวจ pip packages
3. อ่าน console output

ยินดีช่วย! 🎉
