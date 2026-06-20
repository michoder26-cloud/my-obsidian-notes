# Setup Positions Sheet for AI Reflection

## วิธี 1: Manual Copy (เร็ว)

### Step 1: ใน Google Sheet เปิด "Trade" sheet
- Select ทั้งหมด (Ctrl+A)
- Copy (Ctrl+C)

### Step 2: สร้าง/เปิด "Positions" sheet
- ถ้าไม่มี → Insert > Sheet ตั้งชื่อ "Positions"
- ถ้ามี → เอา old data ออก

### Step 3: วาง data
- Paste (Ctrl+V) จาก Trade sheet

### Step 4: เพิ่ม column "Outcome" (col X)
```
ถ้า row มี outcome + lesson → ใช้ได้เลย
ถ้ายังไม่มี → ใส่ "pending" ไปก่อน
```

---

## วิธี 2: Auto-Copy Script (ดีกว่า)

ใน Google Sheet:
1. Tools > Script editor
2. Paste code นี้:

```javascript
function copyTradeToPositions() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const tradeSheet = ss.getSheetByName('Trade');
  const posSheet = ss.getSheetByName('Positions') || ss.insertSheet('Positions');
  
  // Get all data from Trade
  const data = tradeSheet.getDataRange().getValues();
  
  // Clear Positions
  posSheet.clear();
  
  // Copy data
  posSheet.getRange(1, 1, data.length, data[0].length).setValues(data);
  
  // Add Outcome column header if missing
  const headers = data[0];
  if (!headers.includes('Outcome')) {
    posSheet.getRange(1, headers.length + 1).setValue('Outcome');
  }
  
  SpreadsheetApp.getUi().alert('✅ Copied Trade → Positions');
}
```

3. Save > เลือก function > Run
4. Authorize access

---

## ผลที่ได้:

**Positions sheet จะมี:**
```
date | action | ticker | confidence | entry | sl | tp | ... | outcome | lesson
2026-05-10 | HOLD | XAUUSD | 0.45 | 2450.50 | ... | ... | pending | 
2026-05-10 | HOLD | XAUUSD | 0.35 | 2450.50 | ... | ... | pending | 
...
```

✅ พอ Reflection Agent run Sunday → จะ evaluate + generate lessons!

---

## Workflow ตอนนี้:

**Mon-Fri (เทรดทุกวัน):**
- Read Lessons → Fetch Data → Bull/Bear/Quant → Judge → **Save ลง Trade sheet**

**Sunday 9PM (สรุปสัปดาห์):**
- Read Positions → Evaluate → Reflection → **Save บทเรียน ลง Lessons sheet**

