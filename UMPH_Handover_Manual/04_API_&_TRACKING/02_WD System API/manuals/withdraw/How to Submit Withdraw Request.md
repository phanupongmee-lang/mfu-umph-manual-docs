---
type: manual
system: reg-withdraw
aliases: [Submit Withdraw, ยื่นถอนรายวิชา]
tags: [guide, workflow]
created: 2026-03-30
---

# 📖 How to Submit Withdraw Request

> [!abstract] Overview (ภาพรวม)
> คู่มือนี้อธิบายขั้นตอนการที่นักศึกษายื่นคำร้องขอถอนรายวิชาผ่าน API ตั้งแต่การตรวจสอบ JWT จนถึงการรับผลการยื่น  
> ใช้เมื่อ: นักศึกษาต้องการถอนรายวิชา หรือ developer ทดสอบ submit flow

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มี MISAPI JWT Token (ได้จาก `/tokencheck` หรือ login portal)
- [ ] ทราบ `studentcode`, `acadyear`, `semester`
- [ ] ทราบ `classid` และ `courseid` ของรายวิชาที่ต้องการถอน
- [ ] ระบบปฏิทินเปิดรับคำร้อง (ตรวจสอบจาก `/staff/checkcalendar`)

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ตรวจสอบ / แปลง JWT Token** (ถ้ามาจาก Portal ภายนอก)
   ```http
   GET /reg/withdraw/tokencheck
   token: <external_jwt>
   ```
   - รับ `new_token.token` มาใช้ใน header `Authorization: Bearer <token>`

2. **ดึงรายวิชาที่สามารถถอนได้**
   ```http
   GET /reg/withdraw/students/studentcourse?acadyear=2567&semester=1
   Authorization: Bearer <token>
   studentcode: 6501234
   ```
   - ผลลัพธ์คือรายการ classid/courseid ที่ถอนได้ในเทอมนั้น

3. **ยื่นคำร้องถอนรายวิชา**
   ```http
   POST /reg/withdraw/students/studentwithdraw
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "data": [
       {
         "studentcode": "6501234",
         "acadyear": 2567,
         "semester": 1,
         "classid": 1001,
         "courseid": "MFU101",
         "reason": "ไม่สามารถเข้าเรียนได้",
         "mobile": "0812345678",
         "email": "student@lamduan.mfu.ac.th"
       }
     ]
   }
   ```
   - API จะตรวจสอบกฎ min-course ก่อนบันทึก
   - หากสำเร็จ: ส่ง email แจ้งนักศึกษาทันที
   - ผลลัพธ์ HTTP 200 + `code: 200`

4. **ติดตามผลการพิจารณา**
   ```http
   GET /reg/withdraw/students/result?acadyear=2567&semester=1
   Authorization: Bearer <token>
   studentcode: 6501234
   ```
   - ดู status แต่ละ wid

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] กฎ Min-Course — HTTP 401
> หากส่ง `data` ที่มีจำนวนรายวิชาเท่ากับหรือมากกว่าจำนวนรายวิชาทั้งหมดของนักศึกษา API จะปฏิเสธด้วย HTTP 200 แต่ `code: 401`:
> ```json
> { "code": 401, "message": "Student must study at least 1 course (can not withdrawn all courses)." }
> ```
> **แก้ไข:** ลดจำนวนรายวิชาที่ต้องการถอน ให้เหลืออย่างน้อย 1 วิชา

> [!warning] Email ส่งต่อแม้ถอนบางวิชาไม่สำเร็จ
> `studentwithdraw` วน loop ต่อรายการ — ถ้า wid แรกสำเร็จแล้ว email ถูกส่งไปแล้ว แม้ wid ถัดไปจะ fail  
> **แก้ไข:** ตรวจสอบ response `code` ของแต่ละรายการก่อน conclude ผล

## 🔗 Related

- [[Withdraw Request]] — โครงสร้างคำร้องและ status codes
- [[Withdraw Calendar]] — การตรวจสอบว่าปฏิทินเปิดอยู่หรือไม่
- [[Withdraw Token Check]] — วิธีแปลง JWT จาก portal ภายนอก

Source: `misapi/reg/withdraw/routes.py`
