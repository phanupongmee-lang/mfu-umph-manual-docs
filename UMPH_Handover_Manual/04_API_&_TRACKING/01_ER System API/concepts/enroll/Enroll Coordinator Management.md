---
type: concept
system: reg-enroll
aliases: [Coordinator, Class Adjust, ผู้ประสานงาน]
tags: [concept]
created: 2026-03-30
---

# Enroll Coordinator Management

## What

Coordinator จัดการ **supply side** ของการลงทะเบียน — ควบคุมที่นั่ง, อนุมัติการแจ้งความประสงค์, และปรับ section ให้นักศึกษา

**Blueprint:** `reg_coordinator = Blueprint('reg/enroll/coordinator', __name__)`  
**Package:** `MIS_API_REG_Enroll_Coordinator`

### 1. การปรับจำนวนที่นั่ง (Class Seat Adjustment)

| Endpoint | Method | Procedure | หน้าที่ |
|---------|--------|-----------|---------|
| `/classadjustresult` | GET | `CLASS_ADJUST_SEL` | ดูสถานะที่นั่งปัจจุบัน |
| `/saveclassadjustresult` | PUT | `CLASS_ADJUST_UPD_LIST` | อัปเดต totalsumseat หลายห้องพร้อมกัน |

Parameters ของ PUT:
```json
{
  "classid_list": [1001, 1002, 1003],
  "totalsumseat_list": [50, 45, 30],
  "ipaddress": "...",
  "userid": "..."
}
```

### 2. การตรวจสอบผล/อนุมัติการแจ้งความประสงค์

| Endpoint | Procedure | หน้าที่ |
|---------|-----------|---------|
| `/checkapprovebycoursesel` GET | `CHECKAPPROVE_BYCOURSE_SEL` | ผลอนุมัติ by course + section |
| `/checkapprovebystusel` GET | `CHECKAPPROVE_BYSTU_SEL` | ผลอนุมัติ by classid + period |

Filter parameters: `levelid, courseid, section, period, userid`

### 3. การย้ายตอนเรียนในการแจ้งความประสงค์

ดูรายละเอียดใน [[Enroll Section Change]]

### 4. Sub-module Staff admin

**Blueprint:** `reg_staff = Blueprint('reg/enroll/staff', __name__)`

| Endpoint | Procedure | หน้าที่ |
|---------|-----------|---------|
| GET/PUT `/configcredit` | `SYSCONFIG_CREDIT_SEL/UPD` | ตั้งค่าหน่วยกิตสูงสุดสำหรับนักศึกษาวิทยาทัณฑ์ (3 ระดับ) |
| GET `/adjustconditionlog` | `ADJUSTCONDITION_LOG` | ประวัติการยกเลิกเงื่อนไข |
| GET `/adjustconditionlogdtl` | `ADJUSTCONDITION_LOG_DETAIL` | รายละเอียดการยกเลิกรายบุคคล |
| POST `/clearpreregdata` | `ClearPreregData` | ล้างข้อมูลการแจ้งความประสงค์ทั้งเทอม |

> [!warning] `/clearpreregdata`
> เป็น destructive operation ที่ล้างข้อมูล pre-reg ทั้งเทอม ต้องใช้ด้วยความระมัดระวัง

## Why

การแยก coordinator blueprint ออกจาก student blueprint ทำให้ access control ชัดเจน  
`totalsumseat_list` รับเป็น Oracle Collection ทำให้ update หลายห้องใน 1 call ลด round trip

## Related

- [[Enroll Pre-Enrollment]] — coordinator อนุมัติรายการจาก pre-enroll
- [[Enroll Section Change]] — bulk section reassignment
- [[How to Adjust Class Seats]] — คู่มือปรับที่นั่ง
- [[ADR-003 Multi-Blueprint Enroll Sub-Modules]] — เหตุผลแยก Blueprint

Source: `misapi/reg/enroll/coordinator/routes.py`, `misapi/reg/enroll/coordinator/data_access.py`, `misapi/reg/enroll/staff/routes.py`
