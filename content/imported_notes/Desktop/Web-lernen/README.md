# เว็บสอน Python เบื้องต้น

เว็บสอน Python สำหรับผู้เริ่มต้น มี 7 บท + แบบทดสอบ + code playground (รัน Python ในเบราว์เซอร์)

## โครงสร้าง

```
Web-lernen/
├── backend/         # Node.js + Express — เสิร์ฟบทเรียน + ตรวจ quiz
│   ├── server.js
│   ├── package.json
│   └── data/
│       ├── lessons.json
│       └── quizzes.json
└── frontend/        # React (Vite) + Pyodide
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── App.jsx
        ├── api.js          # เรียก backend
        ├── pyodide.js      # โหลด/รัน Python
        ├── main.jsx
        ├── styles.css
        └── components/
            ├── Sidebar.jsx
            ├── Lesson.jsx
            ├── CodePlayground.jsx
            └── Quiz.jsx
```

## วิธีรัน

ต้องติดตั้ง **Node.js 18+** ก่อน

### 1. รัน backend (terminal ที่ 1)

```powershell
cd backend
npm install
npm run dev
```

ขึ้น `Backend running on http://localhost:3001` = พร้อม

### 2. รัน frontend (terminal ที่ 2)

```powershell
cd frontend
npm install
npm run dev
```

เปิด `http://localhost:5173` ในเบราว์เซอร์

> ครั้งแรกที่กด Run โค้ด Python — Pyodide จะดาวน์โหลด ~10MB จาก CDN ใช้เวลา 5–15 วินาที หลังจากนั้นรันโค้ดเร็วทันที

## การทำงานของระบบ

```
   ┌──────────────┐                         ┌──────────────┐
   │   Browser    │                         │   Backend    │
   │  (React UI)  │                         │  (Express)   │
   └──────┬───────┘                         └──────┬───────┘
          │                                        │
          │  GET /api/lessons                      │
          │ ───────────────────────────────────►   │
          │  ◄────── [{id,title,order}]            │
          │                                        │
          │  GET /api/lessons/intro                │
          │ ───────────────────────────────────►   │
          │  ◄────── เนื้อหาบท + ตัวอย่างโค้ด         │
          │                                        │
          │  POST /api/quiz/intro/check {answers}  │
          │ ───────────────────────────────────►   │
          │  ◄────── {score, results, เฉลย}         │
          │                                        │
   ┌──────▼──────────────┐
   │  Pyodide (ในบราวเซอร์) │   ◄── รันโค้ด Python ที่นี่
   │  (WebAssembly)       │      ไม่ส่งโค้ดไปที่ server
   └──────────────────────┘
```

### จุดสำคัญ

1. **Backend ไม่รันโค้ด Python** ของผู้ใช้ — โค้ดถูกรันใน browser ด้วย Pyodide (Python ที่ compile เป็น WebAssembly)
   - **ปลอดภัย**: ไม่ต้องทำ sandbox บน server
   - **เร็ว**: ไม่มี network round-trip ตอนรันโค้ด

2. **Backend ทำหน้าที่:**
   - เสิร์ฟเนื้อหาบทเรียน (`/api/lessons`, `/api/lessons/:id`)
   - ส่ง quiz **โดยซ่อนเฉลย** (`/api/quiz/:id`) — ผู้ใช้ดูเฉลยจาก devtools ไม่ได้
   - ตรวจคำตอบและส่งเฉลย (`/api/quiz/:id/check`)

3. **เพิ่ม/แก้บทเรียน** — แก้ไฟล์ `backend/data/lessons.json` และ `quizzes.json` แล้ว reload ได้เลย ไม่ต้อง restart frontend

## ปรับแต่งต่อ

- **เพิ่มบทเรียนใหม่**: เพิ่ม object ใน `lessons.json` พร้อม `id` และ `order` ใหม่ — เพิ่ม quiz ที่ key เดียวกันใน `quizzes.json`
- **เก็บ progress ผู้เรียน**: เพิ่ม endpoint `/api/progress` + ใช้ `localStorage` ฝั่ง client หรือฐานข้อมูลฝั่ง server
- **deploy**: build frontend ด้วย `npm run build` แล้วให้ Express เสิร์ฟไฟล์จาก `frontend/dist/`
