---
type: manual
system: reg-withdraw
aliases: [Approve Withdraw Instructor, อนุมัติถอนรายวิชาอาจารย์]
tags: [guide, workflow]
created: 2026-03-30
---

# 📖 How to Approve Withdraw as Instructor

> [!abstract] Overview (ภาพรวม)
> คู่มือนี้อธิบายขั้นตอนการที่อาจารย์ผู้สอนตรวจสอบและอนุมัติ/ปฏิเสธคำร้องถอนรายวิชาของนักศึกษาในกลุ่มเรียนที่รับผิดชอบ  
> ใช้เมื่อ: อาจารย์ผู้สอนต้องการพิจารณาคำร้องถอน หรือ developer integrate instructor approval flow

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มี MISAPI JWT Token
- [ ] ทราบ `officerid` ของอาจารย์ (ใช้ใน header)
- [ ] มีสิทธิ์ Role Instructor ใน `authen_roles`
- [ ] มีคำร้องถอนที่รออนุมัติ (status = null หรือ W)

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ดูรายการคำร้องที่รออนุมัติ**
   ```http
   GET /reg/withdraw/instructor/allstudentrequest?officerstatus=
   Authorization: Bearer <token>
   officerid: EMP001
   ```
   - `officerstatus` ส่งค่าว่างเพื่อดูรายการใหม่ที่ยังไม่อนุมัติ
   - `A` = ดูที่อนุมัติแล้ว, `N` = ดูที่ปฏิเสธแล้ว

2. **ดูรายละเอียดคำร้องตาม classid/courseid**
   ```http
   POST /reg/withdraw/instructor/studentrequestdetail
   Authorization: Bearer <token>
   officerid: EMP001
   Content-Type: application/json

   {
     "officerstatus": "",
     "classid": 1001,
     "section": 1,
     "courseid": "MFU101"
   }
   ```

3. **อนุมัติ / ปฏิเสธ คำร้อง (batch by wid)**
   ```http
   PUT /reg/withdraw/instructor/studentrequest
   Authorization: Bearer <token>
   officerid: EMP001
   Content-Type: application/json

   {
     "data": [
       { "wid": 1001, "approvestatus": "A", "comments": "อนุมัติ" },
       { "wid": 1002, "approvestatus": "N", "comments": "ควรเรียน" }
     ]
   }
   ```
   - `approvestatus`: `A` = อนุมัติ, `N` = ไม่อนุมัติ, `W` = ขอเรียกพบ
   - หลัง PUT สำเร็จ: ระบบส่ง email แจ้งนักศึกษาทุกรายที่ได้รับการพิจารณา

4. **ดูประวัติการอนุมัติ (Log)**
   ```http
   GET /reg/withdraw/instructor/historylog
   Authorization: Bearer <token>
   officerid: EMP001
   ```

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] Email ส่งทุกครั้งที่ PUT สำเร็จ
> ระบบส่ง email ทันทีหลังทุก `wid` ที่อนุมัติ หากเรียก PUT ซ้ำจะมี email ส่งซ้ำ  
> **แก้ไข:** ตรวจสอบว่า wid ที่ส่งยังไม่เคยถูกอนุมัติก่อนส่ง request

> [!warning] Position ในอีเมลถูกกำหนดเป็น "1" เสมอ
> Email ที่ส่งหลัง Instructor อนุมัติจะแสดง position = "Lecturer" เสมอ ไม่ขึ้นกับ payload  
> ถ้าต้องการเปลี่ยน label ต้องแก้ไขใน `mail()` function

## 🔗 Related

- [[Withdraw Approval Workflow]] — ภาพรวมลำดับการอนุมัติ 3 ระดับ
- [[Withdraw Email Notification]] — รายละเอียด email ที่ส่งหลังอนุมัติ
- [[Withdraw Request]] — status codes ที่ใช้ใน approvestatus

Source: `misapi/reg/withdraw/routes.py`
