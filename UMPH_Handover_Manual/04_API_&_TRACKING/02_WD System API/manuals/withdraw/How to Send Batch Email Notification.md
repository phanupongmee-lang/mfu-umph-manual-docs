---
type: manual
system: reg-withdraw
aliases: [Batch Email, Send Email Notification, ส่งอีเมลแจ้งเตือน]
tags: [guide, workflow]
created: 2026-03-30
---

# 📖 How to Send Batch Email Notification

> [!abstract] Overview (ภาพรวม)
> คู่มือนี้อธิบายขั้นตอนการที่เจ้าหน้าที่ทะเบียนส่ง batch email เพื่อเตือนอาจารย์/ที่ปรึกษา/คณบดีที่มีคำร้องค้างการพิจารณา และแจ้งนักศึกษาเกี่ยวกับผลการถอน  
> ใช้เมื่อ: ต้องการ remind อาจารย์ก่อน due date หรือแจ้งผลนักศึกษาเป็น batch

## ✅ Prerequisites (สิ่งที่ต้องเตรียมก่อนเริ่มงาน)

- [ ] มีสิทธิ์เจ้าหน้าที่ทะเบียน (Staff)
- [ ] DB มีข้อมูล DUEDATE ใน `SEND_AUTO_EMAIL` procedure
- [ ] SMTP Gmail account `coursewithdraw.reg@mfu.ac.th` ใช้งานได้
- [ ] (หมายเหตุ: Endpoint เหล่านี้ไม่มี JWT authentication)

## ⚙️ Step-by-Step Guide (ขั้นตอนการทำงาน)

1. **ส่ง Batch Email ให้อาจารย์ผู้สอน / ที่ปรึกษา / คณบดี**
   ```http
   POST /reg/withdraw/staff/sendmail
   ```
   - ระบบดึงข้อมูลจาก `SEND_AUTO_EMAIL` 3 รอบ (type 1, 2, 3)
   - **type 1** — Instructor: ส่ง email บอก "มี X นักศึกษารอการพิจารณา ภายใน [DUEDATE]"
   - **type 2** — Advisor: ส่ง email เช่นเดียวกัน
   - **type 3** — Dean: ส่ง email เช่นเดียวกัน
   - ใช้ `mail_teacher()` function (ไม่ใช่ `mail()`)

2. **ส่ง Batch Email ให้นักศึกษา**
   ```http
   POST /reg/withdraw/staff/sendmailstudent
   ```
   - ระบบดึงข้อมูลจาก `SEND_AUTO_EMAIL` type 4
   - ใช้ `mail_student()` function
   - แจ้งนักศึกษาเกี่ยวกับสถานะคำร้องรวมถึง due date

3. **ตรวจสอบผลการส่ง**
   - Response: `{ "code": 200, "message": { "en": "success" } }`
   - หาก SMTP ล้มเหลว → `code: 500`

**โครงสร้าง `mail_teacher()` :**
```python
mail_teacher(
    to=i['OFF_EMAIL'],
    subject='Notification for course withdraw',
    ccode=i['COURSECODE'],
    cname=i['COURSENAMEENG'],
    offname='',
    offnameeng='',
    stdamt=i['COUNTOFSTUDENT'],
    ttype="1",         # 1=instructor, 2=advisor, 3=dean
    duedate=i['DUEDATE']
)
```

## ⚠️ Troubleshooting & Pitfalls (ข้อควรระวัง/ปัญหาที่พบบ่อย)

> [!warning] Authentication ถูก Disabled
> `/staff/sendmail` และ `/staff/sendmailstudent` ไม่มี `@jwt_required`  
> ใครก็ได้ที่รู้ URL สามารถ trigger batch email ได้ — ควรเปิด authentication ใน production

> [!warning] SMTP Credentials Hardcoded
> `coursewithdraw.reg@mfu.ac.th` + App Password อยู่ใน `routes.py` โดยตรง  
> หาก App Password หมดอายุหรือถูก revoke ทุก email function จะหยุดทำงาน  
> **แก้ไข:** ย้าย credentials ไป environment variable + ตรวจสอบอายุ App Password เป็นประจำ

> [!warning] ไม่มี retry mechanism
> หาก SMTP timeout ใน loop อีเมลบางรายการอาจไม่ถูกส่ง แต่ response ยังเป็น 200  
> **ตรวจสอบ:** ดู server log หรือ SMTP error ใน exception handler

## 🔗 Related

- [[Withdraw Email Notification]] — รายละเอียด email template และ positions
- [[Withdraw Staff Management]] — ความสามารถอื่นๆ ของ Staff

Source: `misapi/reg/withdraw/routes.py`
