---
type: manual
system: reg-withdraw
aliases: [Manage Calendar, ตั้งปฏิทินถอนรายวิชา]
tags: [guide, workflow]
created: 2026-03-30
---

# 📖 How to Manage Withdraw Calendar

> [!abstract] Overview (ภาพรวม)
> คู่มือนี้อธิบายขั้นตอนการที่เจ้าหน้าที่ทะเบียนตั้งและตรวจสอบปฏิทินช่วงเปิดรับถอนรายวิชา  
> ใช้เมื่อ: ต้องการเปิด/ปิด ช่วงถอนรายวิชาในเทอมใดเทอมหนึ่ง หรือตรวจสอบว่าปฏิทินยังเปิดอยู่หรือไม่

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มีสิทธิ์เจ้าหน้าที่ทะเบียน (Staff)
- [ ] ทราบ `acadyear` และ `semester` ที่ต้องการตั้งปฏิทิน
- [ ] ทราบวันที่เริ่ม (`startdate`) และวันที่สิ้นสุด (`enddate`) ในรูปแบบที่ DB รองรับ
- [ ] (หมายเหตุ: Endpoint เหล่านี้ไม่มี JWT authentication — ใช้ได้โดยตรง)

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ตรวจสอบปฏิทินปัจจุบัน**
   ```http
   GET /reg/withdraw/staff/checkcalendar
   userid: <userid>
   ```
   - ผลลัพธ์: `{ "data": [{ "vcalendar_on": "Y" }] }` (Y = เปิด, N = ปิด)

2. **ดูข้อมูลปฏิทินของเทอมที่ต้องการ**
   ```http
   GET /reg/withdraw/staff/calendars?acadyear=2567&semester=1
   ```
   - ผลลัพธ์: รายการปฏิทินพร้อม startdate/enddate ที่บันทึกไว้

3. **ดึงรายการ Academic Year ที่ระบบรองรับ**
   ```http
   GET /reg/withdraw/staff/acadyear
   Authorization: Bearer <token>
   ```
   - ผลลัพธ์: acadyear ปัจจุบันและย้อนหลัง 6 ปี (จาก `FUNC_GET_ACADYEAR` Oracle Function)
   - semester: 1, 2, 3

4. **เพิ่ม / อัปเดตปฏิทิน**
   ```http
   POST /reg/withdraw/staff/calendars
   Content-Type: application/json

   {
     "data": [
       {
         "WITHDRAW_CARLENDAR_CODE": "W2567-1",
         "WITHDRAW_CARLENDAR_NAME": "ถอนรายวิชา เทอม 1/2567",
         "WITHDRAW_CARLENDAR_ACADYEAR": 2567,
         "WITHDRAW_CARLENDAR_SEMESTER": 1,
         "WITHDRAW_CARLENDAR_STARTDATE": "01/08/2024",
         "WITHDRAW_CARLENDAR_ENDDATE": "31/08/2024"
       }
     ]
   }
   ```
   - DB: `MIS_API_REG_W_MASTER_PKG.ADD_CALENDAR`
   - Response: `{ "code": 200 }` หากสำเร็จ

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] Authentication ถูก Disabled
> `/staff/checkcalendar`, `/staff/calendars` (GET/POST) ไม่มี `@jwt_required` — ทุกคนที่รู้ URL เรียกได้  
> **แก้ไข:** ควรเปิดใช้ `@jwt_required` กลับในสภาพแวดล้อม production

> [!warning] รูปแบบวันที่
> `WITHDRAW_CARLENDAR_STARTDATE` / `ENDDATE` ต้องตรงกับ format ที่ Oracle DB ต้องการ  
> ตรวจสอบ format กับ DBA — หากวันที่ผิด `ADD_CALENDAR` จะ return error code ≠ 200

## 🔗 Related

- [[Withdraw Calendar]] — โครงสร้างและ stored procedures ของปฏิทิน
- [[Withdraw Staff Management]] — ความสามารถอื่นๆ ของ Staff
- [[ADR-002 Oracle Function vs Procedure]] — เหตุผลใช้ FUNC_GET_ACADYEAR

Source: `misapi/reg/withdraw/routes.py`, `misapi/reg/withdraw/data_access.py`
