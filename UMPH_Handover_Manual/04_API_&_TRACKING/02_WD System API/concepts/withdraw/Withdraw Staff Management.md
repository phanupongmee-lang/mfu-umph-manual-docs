---
type: concept
system: reg-withdraw
aliases: [Staff Management, Withdraw Staff, เจ้าหน้าที่ถอนรายวิชา]
tags: [concept]
created: 2026-03-30
---

# Withdraw Staff Management

## What

เจ้าหน้าที่ (Staff) ดูแลระบบถอนรายวิชาในระดับ administration มีความสามารถ:

### 1. จัดการปฏิทิน (`/staff/calendars`)
- GET: ดึงข้อมูลปฏิทินตาม acadyear/semester
- POST: เพิ่ม/อัปเดตปฏิทินถอนรายวิชา

### 2. จัดการสิทธิ์ (`/staff/permission`)
- GET: ดู staff permission (`MIS_API_REG_W_MASTER_PKG.GET_STAFF_PERMISSION`)
- POST: เพิ่ม/อัปเดตสิทธิ์ (`ADD_STAFF_PERMISSION`)
- GET `/staff/permission/name`: ดูชื่อ staff (`GET_STAFF_NAME`)

Permission data:

| Field | ความหมาย |
|-------|----------|
| `OFFICERID` | รหัสเจ้าหน้าที่ |
| `WITHDRAW_USERGROUPID` | กลุ่มสิทธิ์ |
| `PERMISSION_STATUS` | สถานะสิทธิ์ (active/inactive) |
| `FULLNAME` | ชื่อ-นามสกุล |

### 3. รายงาน (`/staff/courseswithdraw`, `/staff/studentswithdraw`)

| Endpoint | หน้าที่ |
|---------|---------|
| `GET /staff/courseswithdraw` | รายการรายวิชาที่มีคำร้องถอน กรองตาม acadyear/semester/faculty/courseid |
| `GET /staff/studentswithdraw` | รายชื่อนักศึกษาที่ถอนรายวิชา กรองตาม acadyear/semester/courseid |
| `GET /staff/students` (search) | ค้นหานักศึกษา by studentid/studentname |

### 4. ส่ง Batch Email
- `POST /staff/sendmail` — ส่งให้ Instructor (type 1) + Advisor (type 2) + Dean (type 3)
- `POST /staff/sendmailstudent` — ส่งให้นักศึกษา (type 4)

### 5. ข้อความหน้าแรก (Homepage Message)
- GET `/staff/msghome`: ดึงข้อความ (`GET_MSGHP`)
- POST `/staff/msghome`: บันทึกข้อความ (`ADD_MSGHP`)

### 6. ข้อมูลปิด/เปิดระบบ (Permit)
- GET `/staff/permit`: ดูสถานะการเปิดระบบ (`GET_PERMIT`)
- GET `/staff/msgpermit`: ดึงข้อความแจ้งเตือนปิดระบบ (`GET_MSGPERMIT`)

> [!warning] Note
> Staff endpoints หลายจุด (`/staff/calendars`, `/staff/sendmail`, `/staff/sendmailstudent`) มี `@jwt_required` ถูก comment out — **ไม่มีการ authentication**

## Why

การแยก staff management ออกจาก student/instructor path ทำให้สามารถให้สิทธิ์เฉพาะเจ้าหน้าที่ทะเบียนได้  
Batch email จำเป็นเพราะอาจารย์อาจไม่ได้ติดตาม real-time จึงต้องมี reminder พร้อม due date

## Related

- [[Withdraw Calendar]] — รายละเอียดโครงสร้างปฏิทิน
- [[Withdraw Email Notification]] — รายละเอียดการส่ง email
- [[How to Manage Withdraw Calendar]] — คู่มือการตั้งปฏิทิน
- [[How to Send Batch Email Notification]] — คู่มือการส่ง batch email

Source: `misapi/reg/withdraw/routes.py`, `misapi/reg/withdraw/data_access.py`
