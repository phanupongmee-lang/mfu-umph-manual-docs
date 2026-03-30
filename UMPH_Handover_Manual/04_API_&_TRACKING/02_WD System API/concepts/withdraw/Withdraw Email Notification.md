---
type: concept
system: reg-withdraw
aliases: [Email Notification, Withdraw Email, อีเมลแจ้งผล]
tags: [concept]
created: 2026-03-30
---

# Withdraw Email Notification

## What

ระบบส่งอีเมลแจ้งผลการถอนรายวิชาด้วย SMTP Gmail มี 2 ฟังก์ชันหลัก:

### `mail(to, subject, ccode, cname, status_approve, proves, positions)`
ส่งอีเมลถึง **นักศึกษา** เมื่อมีการเปลี่ยนสถานะคำร้อง

| positions | ผู้ดำเนินการ | สถานการณ์ |
|-----------|-------------|-----------|
| `""` (blank) | — | นักศึกษายื่นคำร้องใหม่ |
| `"1"` | Lecturer / อาจารย์ผู้สอน | Instructor อนุมัติ |
| `"2"` | Advisor / อาจารย์ที่ปรึกษา | Advisor อนุมัติ |
| `"3"` | Dean / คณบดี | Dean อนุมัติ by wid |
| `"4"` | Dean / คณบดี | Dean อนุมัติ batch |
| `"0"` | — | นักศึกษายกเลิกคำร้อง (status `C`) |

position `"4"` และ `"0"` ใช้ template พิเศษ (Dean batch / Cancel) ไม่มีชื่อผู้อนุมัติ

### `mail_teacher(to, subject, ccode, cname, offname, offnameeng, stdamt, ttype, duedate)`
ส่งอีเมลถึง **อาจารย์/ที่ปรึกษา/คณบดี** เพื่อแจ้งรายการที่รอดำเนินการ

| ttype | ผู้รับ |
|-------|--------|
| `"1"` | Instructor — ต้องอนุมัติ X นักศึกษา ภายใน DUEDATE |
| `"2"` | Advisor — ต้องอนุมัติ X นักศึกษา ภายใน DUEDATE |
| `"3"` | Dean — ต้องอนุมัติ X นักศึกษา ภายใน DUEDATE |

### SMTP Configuration

```python
gmail_user = 'coursewithdraw.reg@mfu.ac.th'
gmail_app_password = 'wizl nzhp sxex hwzl'   # App Password (Google)
server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
```

> [!warning] Security Note
> Credentials ถูก hardcode ใน routes.py โดยตรง ควรย้ายไป environment variable หรือ config file

### DB call สำหรับ batch email

```python
# ดึงรายชื่อและข้อมูลที่ต้องส่ง
result = db.get_teacher_to_email(type)   # type: 1=instructor, 2=advisor, 3=dean, 4=student
# Proc: MIS_API_REG_W_OFFICER_PKG.SEND_AUTO_EMAIL
```

## Why

การแจ้งผลทันทีผ่านอีเมลลดภาระการ follow-up และทำให้นักศึกษาทราบสถานะโดยเร็ว  
batch email (`/staff/sendmail`) ใช้สำหรับกรณีอาจารย์ยังไม่ได้ตรวจ — ส่งเป็น reminder พร้อม due date

## Related

- [[Withdraw Approval Workflow]] — trigger point ที่เรียก mail()
- [[Withdraw Staff Management]] — staff trigger batch email
- [[How to Send Batch Email Notification]] — คู่มือการส่ง batch email
- [[Withdraw Request]] — status codes ที่ใช้ใน email

Source: `misapi/reg/withdraw/routes.py`
