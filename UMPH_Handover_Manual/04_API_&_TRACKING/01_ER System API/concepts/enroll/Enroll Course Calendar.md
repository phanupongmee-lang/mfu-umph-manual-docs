---
type: concept
system: reg-enroll
aliases: [Enroll Calendar, ปฏิทินลงทะเบียน, Current Calendar]
tags: [concept]
created: 2026-03-30
---

# Enroll Course Calendar

## What

ปฏิทินการลงทะเบียน (Course Calendar) กำหนดว่าช่วงเวลาใดนักศึกษาสามารถแจ้งความประสงค์หรือลงทะเบียนได้ ควบคุมผ่าน `CURRENT_CALENDAR` procedure

**Key parameters ใน `/currentcalendar`:**

| Header | ความหมาย |
|--------|----------|
| `acadyear` | ปีการศึกษา (required, ต้องเป็น int) |
| `semester` | เทอม 1/2/3 (required, ต้องเป็น int) |
| `schedulegroupid` | กลุ่มตารางสอน — ใช้แบ่งนักศึกษาตามระดับ/หน่วยงาน |
| `studentid` | รหัสนักศึกษา |
| `usertype` | ประเภทผู้ใช้ (required) |
| `lang` | ภาษา TH/EN |

**Stored Procedures:**

| Procedure | Package | หน้าที่ |
|-----------|---------|---------|
| `CURRENT_CALENDAR` | `MIS_API_REG_ENROLL_PKG` | ปฏิทินที่เปิดอยู่สำหรับนักศึกษา |
| `GET_ACADYEAR` | `MIS_API_REG_ENROLL_MASTER_PKG` | ดึงรายปีการศึกษาทั้งหมด |
| `GET_SEMESTER` | `MIS_API_REG_ENROLL_MASTER_PKG` | ดึงรายเทอมทั้งหมด |

**`schedulegroupid` field:**
- ใช้แบ่งกลุ่มนักศึกษาที่มีตารางเรียนต่างกัน (เช่น นักศึกษาปกติ vs นักศึกษาพิเศษ)
- ในรายงาน B1: ถ้า `schedulegroupid == None` → ใช้ค่า default `1000`

**`schedule_period`:**
- รอบของการแจ้งความประสงค์ภายในเทอม (1 เทอมอาจมีหลายรอบ)
- ใช้เป็น parameter ใน `PRE_REG_CONFIRM` และ process approval

**Validation ใน routes.py:**
```python
if not acadyear.isnumeric():
    return 403 'acadyear is not int'
if not semester.isnumeric():
    return 403 'semester is not int'
if not usertype:
    return 403 'Missing usertype parameter'
```

## Why

การแบ่งปฏิทินตาม `schedulegroupid` ให้ความยืดหยุ่นในการเปิด/ปิดการลงทะเบียนแตกต่างกันตามกลุ่มนักศึกษา  
`schedule_period` ให้ระบบรองรับการแจ้งความประสงค์หลายรอบในเทอมเดียว

## Related

- [[Enroll Pre-Enrollment]] — ใช้ schedule_period เพื่อ confirm การแจ้งความประสงค์
- [[Enrollment]] — ปฏิทินควบคุมช่วงเวลา enroll จริง
- [[How to Submit Pre-Enrollment]] — วิธีตรวจสอบปฏิทินก่อนแจ้งความประสงค์

Source: `misapi/reg/enroll/routes.py`, `misapi/reg/enroll/data_access.py`
