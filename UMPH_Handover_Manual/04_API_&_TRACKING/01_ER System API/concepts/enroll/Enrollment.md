---
type: concept
system: reg-enroll
aliases: [Enrollment, ลงทะเบียน, ENROLL_PKG]
tags: [concept]
created: 2026-03-30
---

# Enrollment

## What

การลงทะเบียนจริง (Enrollment) คือ **Phase 2** ที่นักศึกษาเพิ่ม/ลบรายวิชาในตารางลงทะเบียนของตน หลังผ่านกระบวนการ Pre-Enrollment แล้ว

**โครงสร้าง CRUD `/enroll`:**

| Method | Procedure | หน้าที่ |
|--------|-----------|---------|
| GET | `ENROLL_SEL` | ดูรายวิชาที่ลงทะเบียนไว้ทั้งหมด |
| POST | `ENROLL_INS` | เพิ่มรายวิชา / ลงทะเบียน |
| DELETE | `ENROLL_DEL` | ลบรายวิชาที่ลงทะเบียน |

**POST `/enroll` — parameters สำคัญ:**

| Field | ความหมาย |
|-------|----------|
| `studentid` | รหัสนักศึกษา |
| `usertype` | ประเภทผู้ใช้ |
| `wauto` | flag ลงทะเบียนอัตโนมัติ |
| `classid` | ID กลุ่มเรียน |
| `action` | action type (เพิ่ม/แก้ไข) |
| `create_userid` | ผู้ดำเนินการ |
| `create_ip` | IP ผู้ดำเนินการ (ดึงอัตโนมัติจาก `utils.getHost()`) |

**`create_ip` tracking:**
```python
create_ip = str(utils.getHost())  # ดึง IP อัตโนมัติ ไม่รับจาก client
```

**Stored Procedures (`MIS_API_REG_ENROLL_PKG`):**

| Procedure | หน้าที่ |
|-----------|---------|
| `ENROLL_SEL` | ดึงรายการลงทะเบียน |
| `ENROLL_INS` | เพิ่มรายวิชา |
| `ENROLL_DEL` | ลบรายวิชา |
| `ENROLL_CONFIRM` | ยืนยันการลงทะเบียน |
| `ENROLLRESULT` | ผลการลงทะเบียน (acadyear/semester ที่ระบุ) |
| `ENROLLLOG` | log ประวัติการลงทะเบียน |
| `ENROLL_CREDIT_SEL` | สรุปหน่วยกิตลงทะเบียน |
| `ENROLLSUMMARY_SEL` | สรุปภาพรวมการลงทะเบียน |
| `SEARCH_STUDENT` | ค้นหานักศึกษา (สำหรับอาจารย์/staff) |
| `ENROLL_INS_CHANGE_SECTION` | เปลี่ยนตอนเรียนใน enrollment |

**Staff/Instructor Search endpoints (`/enroll/search`):** ใช้ `SEARCH_STUDENT` กรองตาม level, faculty, department, program, studentyear, studentcode range, regstatus

## Why

การแยก IP tracking ออกจาก client input ป้องกัน client ส่ง IP ปลอมเข้าระบบ  
`wauto` flag รองรับกรณีที่ระบบลงทะเบียนให้อัตโนมัติ (เช่น ลงทะเบียนตามผลการอนุมัติ pre-enroll)

## Related

- [[Enroll Pre-Enrollment]] — Phase 1 ก่อนมาถึง enrollment
- [[Enroll Confirm]] — การยืนยัน enroll ด้วย ENROLL_CONFIRM
- [[Enroll Section Change]] — เปลี่ยน section หลัง enroll
- [[How to Confirm Enrollment]] — คู่มือ flow ยืนยัน
- [[ADR-004 Two-Phase Enrollment Design]] — เหตุผลการออกแบบ 2 ขั้น

Source: `misapi/reg/enroll/routes.py`, `misapi/reg/enroll/data_access.py`
