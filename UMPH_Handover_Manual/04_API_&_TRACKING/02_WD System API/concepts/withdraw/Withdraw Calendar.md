---
type: concept
system: reg-withdraw
aliases: [Withdraw Calendar, ปฏิทินถอนรายวิชา]
tags: [concept]
created: 2026-03-30
---

# Withdraw Calendar

## What

ปฏิทินถอนรายวิชา (Withdraw Calendar) กำหนดช่วงเวลาที่เปิดรับคำร้องถอน มีโครงสร้าง:

| Field | ความหมาย |
|-------|----------|
| `WITHDRAW_CARLENDAR_CODE` | รหัสปฏิทิน |
| `WITHDRAW_CARLENDAR_NAME` | ชื่อปฏิทิน |
| `WITHDRAW_CARLENDAR_ACADYEAR` | ปีการศึกษา |
| `WITHDRAW_CARLENDAR_SEMESTER` | เทอม (1/2/3) |
| `WITHDRAW_CARLENDAR_STARTDATE` | วันที่เริ่มรับ |
| `WITHDRAW_CARLENDAR_ENDDATE` | วันที่สิ้นสุด |

**Stored Procedures (`MIS_API_REG_W_MASTER_PKG`):**

| Procedure | หน้าที่ |
|-----------|---------|
| `FUNC_GET_ACADYEAR` | Oracle Function — คืนปีการศึกษาปัจจุบัน (scalar) |
| `CHK_CALENDAR` | ตรวจสอบว่าปฏิทินเปิดอยู่หรือไม่ (คืน `vcalendar_on`) |
| `GET_CALENDAR` | ดึงข้อมูลปฏิทินตาม acadyear/semester |
| `ADD_CALENDAR` | เพิ่ม/แก้ไขปฏิทิน |

**การดึง Academic Year — Oracle Function :**

`FUNC_GET_ACADYEAR` เป็น Oracle Function (ไม่ใช่ Procedure) จึงต้องใช้ `call_func`:

```python
def get_acadyear_semester():
    dbase = OracleDb()
    proc = 'MIS_API_REG_W_MASTER_PKG.FUNC_GET_ACADYEAR'
    result = dbase.call_func(proc)   # ← call_func ไม่ใช่ call_proc
    return result
```

ผลลัพธ์ใช้ใน `/staff/acadyear` เพื่อสร้างรายการ acadyear ย้อนหลัง 6 ปี

**Calendar Endpoints:**

| Method | Path | Auth | หน้าที่ |
|--------|------|------|---------|
| GET | `/staff/checkcalendar` | ❌ disabled | ตรวจว่าปฏิทินเปิดหรือไม่ |
| GET | `/staff/calendars` | ❌ disabled | ดึงปฏิทิน |
| POST | `/staff/calendars` | ❌ disabled | เพิ่ม/แก้ไขปฏิทิน |
| GET | `/staff/acadyear` | ✅ jwt | ดึงรายการ acadyear + semester |

> [!warning] Note
> `/staff/checkcalendar` และ `/staff/calendars` มี `@jwt_required` และ `@authen_roles` ถูก comment out ออก — endpoint เหล่านี้ **ไม่มีการ authentication**

## Why

การแยก calendar ออกจาก business logic ทำให้เจ้าหน้าที่ควบคุมช่วงเวลาได้โดยไม่ต้องแก้โค้ด  
`FUNC_GET_ACADYEAR` เป็น Oracle Function ที่ return ค่า scalar โดยตรง ต่างจาก Stored Procedure ที่ใช้ OUT parameter

## Related

- [[Withdraw Staff Management]] — การจัดการสิทธิ์และรายงานโดยเจ้าหน้าที่
- [[How to Manage Withdraw Calendar]] — คู่มือการตั้งปฏิทิน
- [[ADR-002 Oracle Function vs Procedure]] — เหตุผลใช้ call_func

Source: `misapi/reg/withdraw/routes.py`, `misapi/reg/withdraw/data_access.py`
