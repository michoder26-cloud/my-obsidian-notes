# Security Challenges (ฝึก Python + Mindset แบบ Security)

ลำดับการทำ → จากง่ายไปยาก ทำตามลำดับจะดีที่สุด

| # | ไฟล์ | หัวข้อ | ทักษะหลัก |
|---|---|---|---|
| 1 | `01_caesar_cipher.py` | Cryptography เบื้องต้น | string, ord/chr, loop |
| 2 | `02_password_checker.py` | Password Security | string method, condition |
| 3 | `03_hash_cracker.py` | Dictionary Attack | hashlib, file I/O |
| 4 | `04_port_scanner.py` | Network Scanning | socket, networking |

## วิธีใช้

1. เปิดไฟล์ที่ 1 ก่อน
2. อ่าน docstring ด้านบน เข้าใจว่าโจทย์คืออะไร
3. ทำ TODO ทีละข้อ — มี Hint แต่ไม่มีเฉลย
4. รันด้วย `python 01_caesar_cipher.py` ดูผลทดสอบ
5. ถ้าผ่านทุก test ค่อยขยับไปข้อต่อไป
6. ติดตรงไหนค่อยมาถาม — บอกว่าติดตรง TODO ไหน

## กฎสำคัญ

- ⚠️ **ห้าม scan / crack เครื่องของคนอื่น** เด็ดขาด
- ใช้ `127.0.0.1` กับ `scanme.nmap.org` เท่านั้น
- ถ้ารู้สึกอยากลองของจริง → ไป **TryHackMe** หรือ **HackTheBox** (sandbox ถูกกฎหมาย)

## เป้าหมายที่แท้จริงของชุดโจทย์นี้

ไม่ใช่แค่ "ทำให้เสร็จ"
แต่คือ **"ดูว่าตัวเองสนุกกับมันไหม"**

ถ้าทำแล้วรู้สึก:
- 🔥 อยากรู้ต่อ อยากลองเพิ่ม → สาย Security เหมาะกับคุณ
- 😴 น่าเบื่อ ไม่อยากทำ → ลองสายอื่นดู (เช่น Web Dev, Data, AI)

ใช้เวลาประมาณ 1-2 อาทิตย์กับชุดนี้ก่อนตัดสินใจทุ่มสุดตัว
