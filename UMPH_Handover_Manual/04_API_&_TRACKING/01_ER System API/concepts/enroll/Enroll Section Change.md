---
type: concept
system: reg-enroll
aliases: [Section Change, ย้ายตอนเรียน, Change Section]
tags: [concept]
created: 2026-03-30
---

# Enroll Section Change

## What

การย้ายตอนเรียน (Section Change) รองรับทั้งสองระยะ — สามารถย้ายได้ทั้งในขั้น Pre-Enrollment และ Enrollment จริง

### Pre-Enrollment Section Change
**Procedure:** `PRE_REG_INS_CHANGE_SECTION`  
**Endpoint:** ผ่าน coordinator routes  

Parameters: `studentid, acadyear, semester, classid_new, classid_old, create_userid, lang`  
คืน: `[pre_reg_hd_id, pre_reg_dtl_guid, vstatus, vmessage]`

### Enrollment Section Change
**Procedure:** `ENROLL_INS_CHANGE_SECTION`  
**Endpoint:** ผ่าน main enroll routes

Parameters: `studentid, acadyear, semester, classid_new, classid_old, create_userid, create_ip, lang`
คืน: `[vstatus, vmessage]`

### Coordinator-Side Section Change (bulk by student list)

**Package: `MIS_API_REG_Enroll_Coordinator`**

| Flow | Procedure | หน้าที่ |
|------|-----------|---------|
| ดูรายการย้าย (by course) | `CLASS_PRE_CHANGE_BYCOURSE_SEL` | รายวิชาที่ขอย้ายตอน |
| ดูรายละเอียด | `CLASS_PRE_CHANGE_BYCOURSE_DTL` | รายชื่อนักศึกษาในรายวิชา |
| ตรวจสอบก่อนย้าย | `CLASS_PRE_CHANGE_BYCOURSE_CHK` | validate studentid_list |
| บันทึกย้าย | `CLASS_PRE_CHANGE_MOVECLASS` | ย้าย batch (studentid_list) |
| ดูรายบุคคล | `CLASS_PRE_CHANGE_BYSTU_SEL` | ค้นหานักศึกษา |
| รายละเอียดรายบุคคล | `CLASS_PRE_CHANGE_BYSTU_SELDTL` | รายวิชาของนักศึกษา |
| เลือก section ใหม่ | `CLASS_PRE_CHANGE_BYSTU_GETSEC` | ดึง section ที่พร้อม |

**Bulk move parameters:**
```python
# studentid_list เป็น list[] ส่งเป็น Oracle Collection
db.SaveClassPreChangeByStudent(
    studentid_list, classid_old, acadyear, semester,
    levelid, courseid, section, ipaddress, submit_userid, lang
)
```

## Why

การรองรับ section change ทั้ง pre-enroll และ enroll ทำให้ทั้งนักศึกษาและ coordinator มีความยืดหยุ่นในการปรับตาราง  
Bulk move โดย coordinator ลดภาระในการประสานงานกรณีต้องย้ายนักศึกษาหลายคนพร้อมกัน

## Related

- [[Enroll Pre-Enrollment]] — section change ใน phase 1
- [[Enrollment]] — section change ใน phase 2
- [[Enroll Coordinator Management]] — coordinator ดำเนิน bulk section move
- [[How to Adjust Class Seats]] — การจัดการที่นั่งพร้อมกับ section

Source: `misapi/reg/enroll/routes.py`, `misapi/reg/enroll/coordinator/routes.py`, `misapi/reg/enroll/data_access.py`
