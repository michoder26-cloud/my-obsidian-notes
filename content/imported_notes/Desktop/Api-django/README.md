# Todo App - Django + React

ตัวอย่างเว็บง่ายๆ ที่แสดงการเชื่อมต่อระหว่าง React (frontend) และ Django (backend) ผ่าน REST API

## โครงสร้าง

```
Api-django/
├── backend/         # Django REST API (port 8000)
└── frontend/        # React + Vite (port 5173)
```

## วิธีการเชื่อมต่อกัน

```
[React :5173]  ──fetch('http://localhost:8000/api/todos/')──>  [Django :8000]
                          ↑                                          |
                          └──────── JSON response ───────────────────┘
```

- React ใช้ `fetch()` ยิงไปที่ Django API
- Django ส่ง JSON กลับ (ผ่าน Django REST Framework)
- ต้องเปิด CORS ที่ Django เพื่อให้ React (คนละ port) เรียกได้

---

## การติดตั้ง Backend (Django)

```bash
cd backend
python -m venv venv
venv\Scripts\activate          # Windows
pip install django djangorestframework django-cors-headers
python manage.py migrate
python manage.py runserver
```

Backend รันที่ http://localhost:8000

ทดสอบ API: http://localhost:8000/api/todos/

## การติดตั้ง Frontend (React)

```bash
cd frontend
npm install
npm run dev
```

Frontend รันที่ http://localhost:5173

---

## Endpoints

| Method | URL                       | ทำอะไร            |
|--------|---------------------------|-------------------|
| GET    | /api/todos/               | ดูทั้งหมด         |
| POST   | /api/todos/               | เพิ่มใหม่         |
| DELETE | /api/todos/<id>/          | ลบ                |
| PATCH  | /api/todos/<id>/          | แก้ไข (toggle)    |
