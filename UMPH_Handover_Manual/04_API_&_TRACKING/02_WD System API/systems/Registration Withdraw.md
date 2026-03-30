---
type: moc
system: reg-withdraw
---

# Registration Withdraw — Map of Content

ระบบขอถอนรายวิชาออนไลน์ (Online Course Withdraw Request System) — Flask API ที่รองรับทุก Role ในกระบวนการถอนรายวิชา ตั้งแต่การยื่นคำร้องของนักศึกษา ไปจนถึงการอนุมัติโดยคณบดี

## 🏗 Architecture & Overview

```
Blueprint: withdraw  (prefix: reg/withdraw)
│
├── /tokencheck             → แปลง External JWT → MISAPI JWT
├── /students/*             → นักศึกษายื่น/ติดตาม/ยกเลิกคำร้อง
├── /instructor/*           → อาจารย์ผู้สอนอนุมัติ
├── /advisor/*              → อาจารย์ที่ปรึกษาอนุมัติ
├── /dean/*                 → คณบดีอนุมัติ
├── /officer/*              → ข้อมูล officer
└── /staff/*                → เจ้าหน้าที่จัดการปฏิทิน/สิทธิ์/รายงาน/ส่งอีเมล
```

**Approval Flow :**
```
Student Submit
    → Instructor (position "1")
        → Advisor (position "2")
            → Dean by-wid (position "3")  หรือ  Dean batch (position "4")
```

**DB Packages:**

| Package | หน้าที่ |
|---------|---------|
| `MIS_API_REG_W_MASTER_PKG` | ปฏิทิน, สถานะ, สิทธิ์เจ้าหน้าที่, ข้อความหน้าบ้าน |
| `MIS_API_REG_W_STUDENT_PKG` | ข้อมูลนักศึกษา, ยื่นถอน, ผลการถอน, ยกเลิก |
| `MIS_API_REG_W_OFFICER_PKG` | อาจารย์/ที่ปรึกษา/คณบดี/staff ดูและอนุมัติ |
| `MIS_API_REG_MASTER_PKG` | GetFaculty (shared) |

**API Endpoints:**

| Method | Path | หน้าที่ |
|--------|------|---------|
| GET | `/tokencheck` | แปลง External JWT → MISAPI JWT |
| GET | `/students/studentinfo` | ข้อมูลนักศึกษา |
| GET | `/students/studentcourse` | รายวิชาที่ถอนได้ |
| POST | `/students/studentwithdraw` | ยื่นคำร้องถอนรายวิชา |
| GET | `/students/result` | ผลการถอน |
| PUT | `/students/cancelable` | ยกเลิกคำร้อง |
| PUT | `/instructor/studentrequest` | อาจารย์ผู้สอนอนุมัติ |
| PUT | `/advisor/studentrequest` | อาจารย์ที่ปรึกษาอนุมัติ |
| GET | `/dean/allstudentrequest` | คณบดีอนุมัติ (batch by student) |
| PUT | `/dean/studentrequestbywid` | คณบดีอนุมัติ (by wid) |
| GET/POST | `/staff/calendars` | จัดการปฏิทินถอนรายวิชา |
| POST | `/staff/sendmail` | ส่ง batch email ให้อาจารย์/ที่ปรึกษา/คณบดี |
| POST | `/staff/sendmailstudent` | ส่ง batch email ให้นักศึกษา |

## 🧠 Core Concepts

- [[Withdraw Request]] — lifecycle ของคำร้องถอนรายวิชา และกฎ min-course
- [[Withdraw Approval Workflow]] — 3-level approval: Instructor → Advisor → Dean
- [[Withdraw Calendar]] — ปฏิทินช่วงเปิดรับถอน และวิธีดึง Academic Year
- [[Withdraw Email Notification]] — SMTP Gmail, template, positions 1-4
- [[Withdraw Staff Management]] — สิทธิ์เจ้าหน้าที่, รายงาน, batch email
- [[Withdraw Token Check]] — แปลง External JWT → MISAPI JWT

## 📚 Work Manuals & Guides

- [[How to Submit Withdraw Request]] — ขั้นตอนนักศึกษายื่นคำร้องถอนรายวิชา
- [[How to Approve Withdraw as Instructor]] — ขั้นตอนอาจารย์ผู้สอนอนุมัติถอนรายวิชา
- [[How to Manage Withdraw Calendar]] — ขั้นตอนเจ้าหน้าที่ตั้งปฏิทินถอนรายวิชา
- [[How to Send Batch Email Notification]] — ขั้นตอนส่ง batch email แจ้งเตือน

## ⚖️ Key Decisions

- [[ADR-001 Single Blueprint Multi-Role Path]] — ใช้ Blueprint เดียวแบ่ง Role ด้วย URL prefix
- [[ADR-002 Oracle Function vs Procedure]] — ใช้ call_func สำหรับ Oracle Function scalar

## 🚀 Implementation Status

| Component | Status |
|-----------|--------|
| Student endpoints | ✅ Active |
| Instructor endpoints | ✅ Active |
| Advisor endpoints | ✅ Active |
| Dean endpoints | ✅ Active |
| Staff calendar mgmt | ✅ Active (auth disabled) |
| Staff email blast | ✅ Active (auth disabled) |
| Token check | ✅ Active (auth disabled) |

## 📁 Source Files

Code: `misapi/reg/withdraw/`
- `routes.py` — Flask endpoints (~1700 lines)
- `data_access.py` — Oracle DB calls
