---
type: moc
system: reg-enroll
---

# Registration Enroll — Map of Content

ระบบลงทะเบียนเรียน (Online Enrollment System) — Flask API ครอบคลุมกระบวนการตั้งแต่นักศึกษาแจ้งความประสงค์ (Pre-Enrollment) จนถึงการลงทะเบียนจริง พร้อมระบบจัดการของ Coordinator, Staff และ Report

## 🏗 Architecture & Overview

```
Blueprints:
│
├── reg_enroll          'reg/enroll'            → student / master endpoints
├── reg_coordinator     'reg/enroll/coordinator' → coordinator seat & section mgmt
├── enroll_process      'reg/enroll/process'     → approval processing
├── reg_staff           'reg/enroll/staff'       → staff admin & config
└── enroll_report       'reg/enroll/report'      → report generation (B1, etc.)
```

**Two-Phase Enrollment Flow:**
```
Phase 1 — Pre-Enrollment (แจ้งความประสงค์)
  Student POST /preenroll         → PRE_REG_INS
  Student POST /preenrollconfirm  → PRE_REG_CONFIRM
  Coordinator approves            → CHECKAPPROVE_BYCOURSE_SEL + approve

Phase 2 — Enrollment (ลงทะเบียนจริง)
  Student POST /enroll            → ENROLL_INS
  Student POST /enrollconfirm     → ENROLL_CONFIRM
```

**DB Packages:**

| Package | หน้าที่ |
|---------|---------|
| `MIS_API_REG_ENROLL_MASTER_PKG` | Master data: acadyear, semester, current user, menu, stat |
| `MIS_API_REG_ENROLL_PRE_ENROLL` | Pre-enrollment CRUD, confirm, section change |
| `MIS_API_REG_ENROLL_PKG` | Enrollment CRUD, calendar, confirm, summary, search |
| `MIS_API_REG_Enroll_Coordinator` | Class seat adjustment, approve pre-enroll, section reassignment |
| `MIS_API_REG_ENROLL_STAFF` | Credit config (probation), condition adjust log, clear pre-reg |

**API Endpoints (reg/enroll blueprint):**

| Method | Path | หน้าที่ |
|--------|------|---------|
| GET | `/acadyear` | ดึง academic year list |
| GET | `/semester` | ดึง semester list |
| GET | `/currentuser` | ข้อมูลผู้ใช้ปัจจุบัน |
| GET | `/currentcalendar` | ปฏิทินที่เปิดอยู่สำหรับนักศึกษา |
| GET/POST/DELETE | `/preenroll` | CRUD การแจ้งความประสงค์ |
| PUT | `/preenroll` | อัปเดต remark ของ pre-enroll |
| POST | `/preenrollconfirm` | ยืนยันการแจ้งความประสงค์ |
| GET | `/preenroll/approveresult` | ผลการอนุมัติแจ้งความประสงค์ |
| GET | `/preenroll/total` | สรุปหน่วยกิตใน pre-enroll |
| GET/POST/DELETE | `/enroll` | CRUD การลงทะเบียน |
| POST | `/enrollconfirm` | ยืนยันการลงทะเบียน |
| GET | `/enroll/enrollresult` | ผลการลงทะเบียน |
| GET | `/enroll/enrolllog` | log การลงทะเบียน |
| GET | `/enroll/total` | สรุปหน่วยกิตลงทะเบียน |
| GET | `/enroll/summary` | สรุปการลงทะเบียน |

## 🧠 Core Concepts

- [[Enroll Pre-Enrollment]] — กระบวนการแจ้งความประสงค์ก่อนลงทะเบียน
- [[Enroll Course Calendar]] — ปฏิทินลงทะเบียน, schedule_period, schedulegroupid
- [[Enrollment]] — การลงทะเบียนจริง, wauto flag, create_ip tracking
- [[Enroll Confirm]] — การยืนยัน 2 ระยะ (pre-confirm และ enroll-confirm)
- [[Enroll Section Change]] — การย้ายตอนเรียน ทั้ง pre-enroll และ enroll
- [[Enroll Coordinator Management]] — การจัดการที่นั่ง, อนุมัติ, ย้ายตอนเรียน

## 📚 Work Manuals & Guides

- [[How to Submit Pre-Enrollment]] — นักศึกษาแจ้งความประสงค์ขอลงทะเบียน
- [[How to Confirm Enrollment]] — ขั้นตอนยืนยันการลงทะเบียน (2 phases)
- [[How to Adjust Class Seats]] — Coordinator ปรับจำนวนที่นั่ง
- [[How to Generate Enroll Report B1]] — ออกรายงาน B1 (ใบลงทะเบียน)

## ⚖️ Key Decisions

- [[ADR-003 Multi-Blueprint Enroll Sub-Modules]] — แบ่ง Blueprint ตาม sub-module role
- [[ADR-004 Two-Phase Enrollment Design]] — ออกแบบกระบวนการ 2 ขั้น pre-enroll + enroll

## 🚀 Implementation Status

| Component | Status |
|-----------|--------|
| Pre-Enrollment (student) | ✅ Active |
| Enrollment (student) | ✅ Active |
| Coordinator endpoints | ✅ Active |
| Process/Approve endpoints | ✅ Active |
| Staff config endpoints | ✅ Active |
| Report B1 | ✅ Active |

## 📁 Source Files

Code: `misapi/reg/enroll/`
- `routes.py` — main student endpoints (Blueprint `reg/enroll`)
- `data_access.py` — Oracle DB calls for main blueprint
- `coordinator/routes.py` — Blueprint `reg/enroll/coordinator`
- `coordinator/data_access.py`
- `process/routes.py` — Blueprint `reg/enroll/process`
- `process/data_access.py`
- `staff/routes.py` — Blueprint `reg/enroll/staff`
- `staff/data_access.py`
- `report/routes.py` — Blueprint `reg/enroll/report`
- `report/data_access.py`
