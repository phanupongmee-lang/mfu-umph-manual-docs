---
type: concept
system: reg-enroll
aliases: [Pre-Enrollment, แจ้งความประสงค์, Pre-Reg]
tags: [concept]
created: 2026-03-30
---

# Enroll Pre-Enrollment

## What

การแจ้งความประสงค์ขอลงทะเบียน (Pre-Enrollment) คือขั้นตอน **Phase 1** ที่นักศึกษาเลือกรายวิชาที่ต้องการเรียน **ก่อน** กำหนดเวลาลงทะเบียนจริง ให้ Coordinator นำไปจัดสรรที่นั่ง

**โครงสร้างข้อมูลหลัก:**

| Field | ความหมาย |
|-------|----------|
| `pre_reg_hd_id` | Header ID ของกลุ่มแจ้งความประสงค์ |
| `pre_reg_dtl_guid` | GUID ของแต่ละรายการ (ใช้ DELETE / PUT) |
| `studentid` | ID นักศึกษา |
| `classid` | ID กลุ่มเรียนที่ต้องการ |
| `acadyear / semester` | ปี/เทอมที่ขอลงทะเบียน |
| `schedule_period` | รอบ/ช่วงเวลาการแจ้งความประสงค์ |
| `remark` | หมายเหตุ เหตุผลที่ขอ |

**Stored Procedures (`MIS_API_REG_ENROLL_PRE_ENROLL`):**

| Procedure | Method | หน้าที่ |
|-----------|--------|---------|
| `PRE_REG_SEL` | GET `/preenroll` | ดึงรายชื่อที่แจ้งไว้ |
| `PRE_REG_INS` | POST `/preenroll` | เพิ่มรายการแจ้งความประสงค์ (คืน pre_reg_hd_id + guid) |
| `PRE_REG_DEL` | DELETE `/preenroll` | ลบรายการ (ใช้ guid) |
| `PRE_REG_UPD` | PUT `/preenroll` | แก้ไข remark ของรายการ |
| `PRE_REG_CONFIRM` | POST `/preenrollconfirm` | ยืนยันส่ง batch ทั้งหมด |
| `PRE_REG_APPROVE_RESULT` | GET `/preenroll/approveresult` | ผลการอนุมัติจาก coordinator |
| `PRE_REG_CREDIT_SEL` | GET `/preenroll/total` | สรุปหน่วยกิตรวม |
| `PRE_REG_SEL_INSTRUCTOR` | (coordinator) | ดึงรายการพร้อมชื่ออาจารย์ผู้สอน |
| `PRE_REG_INS_CHANGE_SECTION` | (coordinator) | ย้ายตอนเรียนใน pre-enroll |

**Response structure จาก PRE_REG_INS:**
```json
{
  "pre_reg_hd_id": 12345,
  "pre_reg_dtl_guid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "status": 200,
  "message": "..."
}
```

## Why

การแยก Pre-Enrollment ออกจาก Enrollment จริงช่วยให้ระบบรองรับนักศึกษาจำนวนมากในช่วงเวลาเดียวกันได้  
Coordinator สามารถวิเคราะห์ความต้องการก่อน แล้วจัดสรรที่นั่งให้เหมาะสม ก่อนเปิดให้ลงทะเบียนจริง

## Related

- [[Enroll Course Calendar]] — ช่วงเวลา schedule_period ที่ pre-enroll เปิดอยู่
- [[Enroll Confirm]] — การ confirm หลังแจ้งความประสงค์ครบ
- [[Enroll Coordinator Management]] — Coordinator อนุมัติและจัดสรรจาก pre-enroll
- [[Enroll Section Change]] — การย้ายตอนเรียนในขั้นตอน pre-enroll
- [[How to Submit Pre-Enrollment]] — คู่มือการแจ้งความประสงค์
- [[ADR-004 Two-Phase Enrollment Design]] — เหตุผลการออกแบบ 2 ขั้นตอน

Source: `misapi/reg/enroll/routes.py`, `misapi/reg/enroll/data_access.py`
