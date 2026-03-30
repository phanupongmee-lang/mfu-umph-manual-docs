---
type: manual
system: reg-enroll
aliases: [Report B1, รายงาน B1, ออกรายงาน]
tags: [guide, workflow, report]
created: 2026-03-30
---

# 📖 How to Generate Enroll Report B1

> [!abstract] Overview (ภาพรวม)
> คู่มือการออกรายงาน B1 — ใบยืนยันการลงทะเบียนสำหรับนักศึกษา โดยรวม header (ข้อมูลนักศึกษา/ใบแจ้งชำระ) กับ detail (รายวิชาที่ลงทะเบียน)  
> ใช้เมื่อ: ออก report หลังนักศึกษา confirm enrollment ในแต่ละภาคเรียน

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มี MISAPI JWT Token
- [ ] นักศึกษา confirm enrollment เรียบร้อยแล้ว (ผ่าน `/enrollconfirm`)
- [ ] ทราบ `studentid`, `acadyear`, `semester`

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ดึงรายงาน B1**
   ```http
   GET /reg/enroll/report/B1
   Authorization: Bearer <token>
   studentid: 6501234
   acadyear: 2567
   semester: 1
   lang: TH
   ```

2. **ตรวจสอบ response**
   - `dataHeaderResult[0] == 20000` → สำเร็จ มีข้อมูล header และ detail
   - `dataHeaderResult[0] != 20000` → มีปัญหา ดู message ใน response

   **โครงสร้าง response ที่คาดหวัง:**
   ```json
   {
     "header": {
       "studentid": 6501234,
       "studentname": "...",
       "schedulegroupid": 1000,
       ...
     },
     "detail": [
       { "classid": 10050, "coursename": "...", "credit": 3 },
       ...
     ]
   }
   ```

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] schedulegroupid = None จะถูก default เป็น 1000
> Report B1 มีตรรกะใน route: `if schedulegroupid == None: schedulegroupid = 1000`  
> นักศึกษาที่ไม่มี group จะถูกประมวลผลด้วย group ID 1000 (default group)

> [!warning] dataHeaderResult[0] ≠ 20000
> Report ดึงข้อมูล header จาก `db_webreg_student.GetStudentForInvoice` (cross-module import)  
> ถ้า return code ≠ 20000 แสดงว่า WebReg student record ไม่ครบ หรือ enrollment ไม่ได้ confirm  
> **แก้ไข:** ตรวจสอบว่า enrollment ผ่าน `/enrollconfirm` เรียบร้อยแล้ว และ student record อยู่ใน webreg

> [!warning] Cross-module dependency
> Route นี้ import `db_webreg_student` จาก `misapi.reg.webreg` — ถ้า webreg module มีปัญหา B1 จะพัง  
> ตรวจสอบ import statement ใน `misapi/reg/enroll/report/routes.py`

## 🔗 Related

- [[Enrollment]] — enrollment ต้อง confirm ก่อนออก report
- [[How to Confirm Enrollment]] — ขั้นตอน confirm
- [[Enroll Coordinator Management]] — schedulegroupid concept

Source: `misapi/reg/enroll/report/routes.py`
