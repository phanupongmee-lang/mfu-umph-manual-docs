---
type: decision
system: reg-withdraw
date: 2026-03-30
status: accepted
tags: [decision]
---

# ADR-001 Single Blueprint Multi-Role Path

## Context

ระบบถอนรายวิชามีผู้ใช้หลายประเภท (Student, Instructor, Advisor, Dean, Staff, Officer) แต่ละประเภทมี API endpoints ของตนเอง ต้องตัดสินใจว่าจะออกแบบ Blueprint ใน Flask อย่างไร

**ตัวเลือก:**
1. แยก Blueprint ต่อ Role (5-6 blueprints)
2. ใช้ Blueprint เดียว แบ่ง Role ด้วย URL prefix path

## Decision

ใช้ **Blueprint เดียว** (`withdraw = Blueprint('reg/withdraw', __name__)`) และแบ่ง Role ด้วย URL segment:

```python
/reg/withdraw/students/*    # นักศึกษา
/reg/withdraw/instructor/*  # อาจารย์ผู้สอน
/reg/withdraw/advisor/*     # อาจารย์ที่ปรึกษา
/reg/withdraw/dean/*        # คณบดี
/reg/withdraw/staff/*       # เจ้าหน้าที่ทะเบียน
/reg/withdraw/officer/*     # officer
```

## Rationale

- **Code co-location**: ทุก endpoint อยู่ใน `routes.py` ไฟล์เดียว ง่ายต่อการ trace flow
- **Shared logic**: `mail()`, `mail_teacher()`, `db.*` ใช้ร่วมกันได้โดยไม่ต้อง import ข้าม module
- **Role enforcement**: ใช้ `@authen_roles` decorator บน endpoint แทนการแบ่ง Blueprint — ไม่ต้อง maintain Blueprint permission rules แยก
- **ระบบเดิม**: pattern นี้สอดคล้องกับ Blueprint อื่นๆ ใน misapi (เช่น `reg/enroll`)

**Trade-off:** `routes.py` ยาว ~1700 บรรทัด แต่ยังอ่านได้ด้วย URL segmentation ที่ชัดเจน

## Related

- [[Withdraw Request]] — โครงสร้าง request ที่ใช้ร่วมกันข้าม role
- [[How to Submit Withdraw Request]] — ตัวอย่าง URL pattern สำหรับ student

Source: `misapi/reg/withdraw/routes.py`
