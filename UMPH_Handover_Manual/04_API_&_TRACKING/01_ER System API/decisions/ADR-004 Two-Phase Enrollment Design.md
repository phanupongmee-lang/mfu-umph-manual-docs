---
type: decision
system: reg-enroll
adr_id: ADR-004
status: Accepted
date: 2026-03-30
tags: [architecture, enrollment, workflow]
---

# ADR-004: Two-Phase Enrollment Design

## Status
**Accepted**

## Context (บริบท)

การลงทะเบียนในช่วง peak (ต้นภาค) มีนักศึกษาหลายพันคนพยายามเข้าถึงระบบพร้อมกัน ถ้า "first-come, first-served" ตรงๆ จะเกิด race condition ใน seat allocation และโอกาสเรียนรายวิชาที่ต้องการลดลง Coordinator ยังต้องการข้อมูลความต้องการล่วงหน้าเพื่อวางแผน section/seat ก่อนวัน enrollment จริง

## Decision (การตัดสินใจ)

ออกแบบ enrollment เป็น **2 Phase**:

```
Phase 1 — Pre-Enrollment (แจ้งความประสงค์)
  นักศึกษา POST /preenroll (เพิ่มรายวิชา)
         → POST /preenrollconfirm (ยืนยันความประสงค์)
         → Coordinator ดูผล → ปรับ seat → approve

Phase 2 — Enrollment (ลงทะเบียนจริง)
  นักศึกษา POST /enroll (เพิ่มรายวิชาที่อนุมัติ)
         → POST /enrollconfirm (ยืนยันลงทะเบียน)
```

แต่ละ phase มี **Confirm** step เป็น "commit" ของ phase นั้น

## Rationale (เหตุผล)

- **Demand gathering**: Coordinator เห็นความต้องการล่วงหน้า → เปิด section เพิ่ม/จัดสรร seat ได้
- **Reduce contention**: นักศึกษาลงทะเบียน Phase 2 เฉพาะ classid ที่ผ่าน approval → ลด conflict
- **Audit trail**: มี pre_reg_hd_id, pre_reg_dtl_guid ใน Phase 1 และ enrollment record ใน Phase 2
- **Flexibility**: `schedule_period` ใน Phase 1 รองรับ multiple pre-enroll rounds ในภาคเดียวกัน

## Key Design Details

| ด้าน | Phase 1 | Phase 2 |
|------|---------|---------|
| Primary Key | `pre_reg_dtl_guid` (GUID) | `classid` + studentid |
| Confirm endpoint | `POST /preenrollconfirm` | `POST /enrollconfirm` |
| IP tracking | `create_ip` = `utils.getHost()` | `create_ip` = `utils.getHost()` |
| Flag | `schedule_period` (round) | `wauto` (auto/manual) |
| DB Package | `MIS_API_REG_ENROLL_PRE_ENROLL` | `MIS_API_REG_ENROLL_PKG` |

## Consequences (ผลกระทบ)

- (+) Coordinator มีข้อมูลเพียงพอก่อน enrollment จริง
- (+) ลด DB lock contention ในช่วง peak
- (-) Flow ซับซ้อนขึ้น: นักศึกษาต้องทำ 2 รอบ
- (-) ต้องรักษา consistency ระหว่าง pre-enroll ที่อนุมัติแล้วกับ enrollment จริง
- (-) staff ต้องการ `ClearPreregData` สำหรับ reset ระบบถ้า pre-enroll มี data เสีย (destructive operation!)

## 🔗 Related

- [[Enroll Pre-Enrollment]] — Phase 1 details
- [[Enrollment]] — Phase 2 details
- [[Enroll Confirm]] — รายละเอียด confirm mechanism
- [[How to Submit Pre-Enrollment]] — คู่มือ Phase 1
- [[How to Confirm Enrollment]] — คู่มือ Phase 2
- [[ADR-003 Multi-Blueprint Enroll Sub-Modules]] — การตัดสินใจเกี่ยวข้อง

Source: `misapi/reg/enroll/routes.py`, `misapi/reg/enroll/data_access.py`
