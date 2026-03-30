---
type: decision
system: reg-enroll
adr_id: ADR-003
status: Accepted
date: 2026-03-30
tags: [architecture, blueprint, flask]
---

# ADR-003: Multi-Blueprint Architecture for Enroll Sub-Modules

## Status
**Accepted**

## Context (บริบท)

ระบบ enrollment มีผู้ใช้หลายกลุ่ม (นักศึกษา, Coordinator, Admin Staff, Process Officer, Report Viewer) แต่ละกลุ่มมีสิทธิ์เข้าถึงชุด endpoint ที่แตกต่างกัน การรวม logic ทั้งหมดไว้ใน blueprint เดียวจะทำให้ไม่สามารถควบคุม role-based access ได้อย่างละเอียด และ codebase จัดการยาก

## Decision (การตัดสินใจ)

แยก Enroll module ออกเป็น **5 Blueprint** ตาม functional domain:

| Blueprint | Path | ผู้ใช้หลัก |
|-----------|------|------------|
| `reg_enroll` | `/reg/enroll` | นักศึกษา |
| `reg_coordinator` | `/reg/enroll/coordinator` | Coordinator ภาควิชา |
| `enroll_process` | `/reg/enroll/process` | Process Officer |
| `reg_staff` | `/reg/enroll/staff` | Admin Staff |
| `enroll_report` | `/reg/enroll/report` | Report Viewer |

ทุก blueprint ใช้ `@jwt_required()` + `@authen_roles` decorators

## Rationale (เหตุผล)

- **Role separation**: ใช้ `authen_roles` กำหนด allowed roles ต่อ endpoint ได้ชัดเจน
- **Independent routing**: URL path สะท้อน role hierarchy (`/coordinator`, `/staff`, etc.)
- **Clean imports**: sub-module แต่ละตัวมี `routes.py` และ `data_access.py` เป็น pair เสมอ
- **Maintainability**: เพิ่ม/ลบ endpoint ใน sub-module ไม่กระทบ core student flow

## Consequences (ผลกระทบ)

- (+) endpoint ของแต่ละ role อยู่ใน file เดียวกัน ง่ายต่อ audit
- (+) register blueprint ใน `__init__.py` อย่างชัดเจน
- (-) cross-module import จำเป็น (เช่น report import จาก webreg)
- (-) ต้องระวัง URL prefix ไม่ซ้ำกันเมื่อ register ทุก blueprint

## 🔗 Related

- [[Registration Enroll]] — MOC แสดง blueprint ทั้ง 5
- [[ADR-004 Two-Phase Enrollment Design]] — การตัดสินใจเกี่ยวข้อง

Source: `misapi/reg/enroll/` — ดู `__init__.py`, `routes.py`, `coordinator/routes.py`, `staff/routes.py`, `report/routes.py`
