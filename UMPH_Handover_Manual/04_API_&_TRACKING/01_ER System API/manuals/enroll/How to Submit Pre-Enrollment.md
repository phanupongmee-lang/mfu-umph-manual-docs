---
type: manual
system: reg-enroll
aliases: [Submit Pre-Enrollment, แจ้งความประสงค์ลงทะเบียน]
tags: [guide, workflow]
created: 2026-03-30
---

# 📖 How to Submit Pre-Enrollment

> [!abstract] Overview (ภาพรวม)
> คู่มือนี้อธิบายขั้นตอนการที่นักศึกษาแจ้งความประสงค์ขอลงทะเบียน (Phase 1) ตั้งแต่ตรวจสอบปฏิทิน เพิ่มรายวิชา จนถึงการยืนยัน  
> ใช้เมื่อ: นักศึกษาต้องการแสดงความประสงค์ขอลงทะเบียนรายวิชาในรอบ pre-enroll หรือ developer ทดสอบ flow

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มี MISAPI JWT Token
- [ ] ทราบ `studentid`, `acadyear`, `semester`
- [ ] ปฏิทินแจ้งความประสงค์เปิดอยู่ (ตรวจสอบจาก `/currentcalendar`)
- [ ] ทราบ `classid` ของรายวิชาที่ต้องการ

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ตรวจสอบปฏิทินที่เปิดอยู่**
   ```http
   GET /reg/enroll/currentcalendar
   Authorization: Bearer <token>
   acadyear: 2567
   semester: 1
   schedulegroupid: 1
   studentid: 6501234
   usertype: STD
   lang: TH
   ```
   - ผลลัพธ์มี `schedule_period` ที่เปิดอยู่ — ใช้ค่านี้ใน confirm

2. **เพิ่มรายวิชาที่ต้องการ (ทีละรายการ)**
   ```http
   POST /reg/enroll/preenroll
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "studentid": 6501234,
     "acadyear": 2567,
     "semester": 1,
     "classid": 10050,
     "remark": "ต้องการเรียนวิชานี้",
     "create_userid": "6501234",
     "lang": "TH"
   }
   ```
   - บันทึก `pre_reg_dtl_guid` ที่ได้รับ — ใช้สำหรับลบหรือแก้ไขรายการ

3. **ดูรายการที่แจ้งไว้ทั้งหมด**
   ```http
   GET /reg/enroll/preenroll
   Authorization: Bearer <token>
   studentid: 6501234
   acadyear: 2567
   semester: 1
   lang: TH
   ```

4. **ลบรายการที่ไม่ต้องการ (ถ้ามี)**
   ```http
   DELETE /reg/enroll/preenroll
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "pre_reg_dtl_guid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "create_userid": "6501234",
     "lang": "TH"
   }
   ```

5. **ยืนยัน (Confirm) การแจ้งความประสงค์**
   ```http
   POST /reg/enroll/preenrollconfirm
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "studentid": 6501234,
     "acadyear": 2567,
     "semester": 1,
     "schedule_period": 1,
     "user_type": "STD",
     "method": "ONLINE",
     "submit_userid": "6501234",
     "lang": "TH"
   }
   ```

6. **ติดตามผลการอนุมัติจาก Coordinator**
   ```http
   GET /reg/enroll/preenroll/approveresult
   Authorization: Bearer <token>
   studentid: 6501234
   acadyear: 2567
   semester: 1
   lang: TH
   ```

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] Missing Required Parameters → 403
> ทุก endpoint ตรวจสอบ required parameters อย่างเข้มงวด  
> `studentid`, `acadyear`, `semester` ต้องเป็นตัวเลขเท่านั้น (`isnumeric()` check)  
> **แก้ไข:** ตรวจสอบว่าส่งค่าเป็น string ที่เป็นตัวเลข เช่น `"6501234"` ไม่ใช่ `6501234` ใน header

> [!warning] schedule_period ต้องตรงกับปฏิทิน
> `schedule_period` ที่ใส่ใน `/preenrollconfirm` ต้องตรงกับที่ดึงจาก `/currentcalendar`  
> ถ้าไม่ตรง DB จะ return error status

## 🔗 Related

- [[Enroll Pre-Enrollment]] — โครงสร้างข้อมูลและ stored procedures
- [[Enroll Course Calendar]] — วิธีดึง schedule_period ที่ถูกต้อง
- [[Enroll Confirm]] — รายละเอียดการ confirm 2 phase
- [[ADR-004 Two-Phase Enrollment Design]] — เหตุผลการออกแบบ flow นี้

Source: `misapi/reg/enroll/routes.py`
