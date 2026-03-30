---
type: manual
system: reg-enroll
aliases: [Adjust Class Seats, ปรับที่นั่ง]
tags: [guide, workflow, coordinator]
created: 2026-03-30
---

# 📖 How to Adjust Class Seats (Coordinator)

> [!abstract] Overview (ภาพรวม)
> คู่มือสำหรับ Coordinator ในการดูรายชื่อนักศึกษาที่แจ้งความประสงค์ตามรายวิชา แล้วปรับที่นั่งรับนักศึกษาให้ตรงความต้องการ  
> ใช้เมื่อ: Coordinator ต้องดูสรุปจำนวน pre-enrollment และกำหนด seat allocation ในแต่ละ class section

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มี MISAPI JWT Token และ role **Coordinator** (`authen_roles`)
- [ ] ทราบ `acadyear`, `semester` ที่ต้องการจัดการ
- [ ] ช่วง pre-enrollment ปิดแล้ว (ดูข้อมูล pre-enroll ได้ครบ)

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ดูผลรวม pre-enroll แยกตามรายวิชา/section**
   ```http
   GET /reg/enroll/coordinator/classadjustresult
   Authorization: Bearer <token>
   acadyear: 2567
   semester: 1
   lang: TH
   ```
   - ผลลัพธ์มี `classid` และจำนวนนักศึกษาที่แจ้งมาแต่ละ section

2. **ตรวจสอบสถานะการอนุมัติแยกตามรายวิชา**
   ```http
   GET /reg/enroll/coordinator/checkapprovebycoursesel
   Authorization: Bearer <token>
   acadyear: 2567
   semester: 1
   courseid: MIS301
   lang: TH
   ```

3. **ตรวจสอบสถานะการอนุมัติแยกตามนักศึกษา**
   ```http
   GET /reg/enroll/coordinator/checkapprovebystusel
   Authorization: Bearer <token>
   acadyear: 2567
   semester: 1
   studentid: 6501234
   lang: TH
   ```

4. **ปรับที่นั่งรับ (กำหนด totalsumseat ของแต่ละ class)**
   ```http
   PUT /reg/enroll/coordinator/saveclassadjustresult
   Authorization: Bearer <token>
   Content-Type: application/json

   {
     "acadyear": 2567,
     "semester": 1,
     "classid_list": [10050, 10051, 10052],
     "totalsumseat_list": [30, 25, 20],
     "create_userid": "coordinator01",
     "lang": "TH"
   }
   ```
   - `classid_list` และ `totalsumseat_list` ต้องมีจำนวนเท่ากัน
   - DB procedure `CLASS_ADJUST_UPD_LIST` ประมวลผล list ทั้งหมดในครั้งเดียว

5. **ตรวจสอบผลหลังปรับ**
   - เรียก `GET /classadjustresult` อีกครั้งเพื่อยืนยัน

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] classid_list และ totalsumseat_list ต้องมีจำนวนเท่ากัน
> DB procedure รับค่าเป็น parallel list ถ้าจำนวนไม่ตรงกัน procedure จะ error  
> **แก้ไข:** ตรวจสอบ length ของทั้งสอง list ก่อน call API

> [!warning] ปรับที่นั่งน้อยกว่าจำนวน approved แล้ว
> ถ้า totalsumseat < จำนวนที่ approved ไปแล้ว DB จะ handle ตาม business rule  
> ควรปรับก่อนช่วง enrollment จริงเท่านั้น

## 🔗 Related

- [[Enroll Coordinator Management]] — โครงสร้าง coordinator procedures ทั้งหมด
- [[Enroll Pre-Enrollment]] — ข้อมูล pre-enroll ที่ coordinator ดูในขั้นตอนที่ 1-3
- [[Enroll Section Change]] — การย้าย section

Source: `misapi/reg/enroll/coordinator/routes.py`, `misapi/reg/enroll/coordinator/data_access.py`
