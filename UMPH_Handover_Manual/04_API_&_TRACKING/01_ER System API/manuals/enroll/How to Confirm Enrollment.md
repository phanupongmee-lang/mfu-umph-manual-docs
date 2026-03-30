---
type: manual
system: reg-enroll
aliases: [Confirm Enrollment, ยืนยันลงทะเบียน]
tags: [guide, workflow]
created: 2026-03-30
---

# 📖 How to Confirm Enrollment

> [!abstract] Overview (ภาพรวม)
> คู่มือนี้อธิบายขั้นตอนการลงทะเบียนและยืนยัน (Phase 2) หลังจาก pre-enrollment ผ่านการอนุมัติแล้ว  
> ใช้เมื่อ: นักศึกษาต้องการลงทะเบียนจริงในช่วงเวลาที่กำหนด หรือ developer ทดสอบ enrollment flow

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มี MISAPI JWT Token
- [ ] Pre-Enrollment ผ่านการอนุมัติจาก Coordinator แล้ว (ตรวจสอบที่ `/preenroll/approveresult`)
- [ ] ปฏิทินลงทะเบียนจริงเปิดอยู่
- [ ] ทราบ `classid` ของรายวิชาที่อนุมัติแล้ว

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ดูผลอนุมัติ Pre-Enrollment**
   ```http
   GET /reg/enroll/preenroll/approveresult
   Authorization: Bearer <token>
   studentid: 6501234
   acadyear: 2567
   semester: 1
   lang: TH
   ```
   - ตรวจสอบว่า classid ที่ต้องการผ่านการอนุมัติแล้ว

2. **เพิ่มรายวิชาเข้า enrollment**
   ```http
   POST /reg/enroll/enroll
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "studentid": 6501234,
     "usertype": "STD",
     "wauto": "N",
     "classid": 10050,
     "action": 1,
     "create_userid": "6501234",
     "lang": "TH"
   }
   ```
   - `wauto`: `"N"` = manual, `"Y"` = auto-enroll
   - `create_ip` ถูกดึงอัตโนมัติ ไม่ต้องส่ง

3. **ดูรายการลงทะเบียนทั้งหมด**
   ```http
   GET /reg/enroll/enroll
   Authorization: Bearer <token>
   studentid: 6501234
   lang: TH
   ```

4. **ดูสรุปหน่วยกิต**
   ```http
   GET /reg/enroll/enroll/total
   Authorization: Bearer <token>
   studentid: 6501234
   acadyear: 2567
   semester: 1
   ```

5. **ยืนยัน (Confirm) การลงทะเบียน**
   ```http
   POST /reg/enroll/enrollconfirm
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "studentid": 6501234,
     "acadyear": 2567,
     "semester": 1,
     "user_type": "STD",
     "submit_userid": "6501234",
     "lang": "TH"
   }
   ```
   - `submit_ip` ถูกดึงอัตโนมัติจาก `utils.getHost()`

6. **ตรวจสอบผลการลงทะเบียน**
   ```http
   GET /reg/enroll/enroll/enrollresult?acadyear=2567&semester=1
   Authorization: Bearer <token>
   studentid: 6501234
   lang: TH
   ```

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] ลงทะเบียนก่อนหน้าต่ออนุมัติ
> ถ้า POST `/enroll` โดยที่ classid ยังไม่ผ่าน pre-enroll approve DB จะ return error status  
> **แก้ไข:** ตรวจสอบ `/preenroll/approveresult` ก่อน POST `/enroll`

> [!warning] wauto flag ส่งผลต่อ business logic ใน DB
> `wauto = "Y"` บอก DB ว่าเป็นการ enroll อัตโนมัติ — DB อาจ bypass บางเงื่อนไข  
> ใช้ `"Y"` เฉพาะเมื่อ coordinator/staff ดำเนินการแทนนักศึกษา

## 🔗 Related

- [[Enrollment]] — โครงสร้าง CRUD enrollment
- [[Enroll Confirm]] — รายละเอียด confirm procedure parameters
- [[Enroll Pre-Enrollment]] — Phase 1 ที่ต้องทำก่อน
- [[How to Generate Enroll Report B1]] — ออกใบยืนยันหลัง confirm

Source: `misapi/reg/enroll/routes.py`
