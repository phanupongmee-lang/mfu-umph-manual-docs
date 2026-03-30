---
type: concept
system: reg-withdraw
aliases: [Approval Workflow, การอนุมัติถอนรายวิชา]
tags: [concept]
created: 2026-03-30
---

# Withdraw Approval Workflow

## What

กระบวนการอนุมัติถอนรายวิชาเป็นแบบ 3-level sequential approval:

```
นักศึกษายื่น (POST /students/studentwithdraw)
        ↓
Instructor / อาจารย์ผู้สอน  (PUT /instructor/studentrequest)     position = "1"
        ↓
Advisor / อาจารย์ที่ปรึกษา  (PUT /advisor/studentrequest)        position = "2"
        ↓
Dean / คณบดี
  ├── อนุมัติรายบุคคล   (PUT /dean/studentrequestbywid)          position = "3"
  └── อนุมัติ batch      (GET /dean/allstudentrequest + body)     position = "4"
```

**approve status codes ที่ส่งใน payload:**

| Code | ความหมาย |
|------|----------|
| `A` | อนุมัติ — Approve |
| `N` | ไม่อนุมัติ — Not Approve |
| `W` | ขอเรียกพบ — Wait / Call meeting |
| `C` | ยกเลิก — Cancel (เฉพาะนักศึกษา) |

**Officer Role → DB Package Mapping:**

| Role | Package | Stored Proc |
|------|---------|-------------|
| Instructor | `MIS_API_REG_W_OFFICER_PKG` | `INSTRUCTOR_APPROVE_BY_WID` |
| Advisor | `MIS_API_REG_W_OFFICER_PKG` | `ADVISOR_APPROVE_BY_WID` |
| Dean (by student) | `MIS_API_REG_W_OFFICER_PKG` | `DEAN_APPROVE_REQUEST` |
| Dean (by wid) | `MIS_API_REG_W_OFFICER_PKG` | `DEAN_APPROVE_REQUEST_BY_ROW` |

**Request/Response pattern สำหรับ approve endpoints:**

```json
// Body
{
  "data": [
    { "wid": 1001, "approvestatus": "A", "comments": "ตกลง" }
  ]
}
// Header
{ "officerid": "EMP001" }
```

แต่ละ approve call จะส่ง email แจ้งผลให้นักศึกษาทันที ผ่าน `mail()` function

## Why

Sequential approval ทำให้ข้อมูลมีลำดับชัดเจน สามารถตรวจสอบได้ว่าใครอนุมัติขั้นใด  
Position code ใช้ระบุตำแหน่งในอีเมลเพื่อแสดง label ที่ถูกต้อง (Lecturer / Advisor / Dean)

## Related

- [[Withdraw Request]] — โครงสร้าง wid และ status codes
- [[Withdraw Email Notification]] — email ที่ส่งหลังทุก approve action
- [[How to Approve Withdraw as Instructor]] — คู่มืออนุมัติของอาจารย์
- [[ADR-001 Single Blueprint Multi-Role Path]] — เหตุผลการออกแบบ URL

Source: `misapi/reg/withdraw/routes.py`
