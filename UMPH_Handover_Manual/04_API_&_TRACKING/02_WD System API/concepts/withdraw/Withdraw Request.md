---
type: concept
system: reg-withdraw
aliases: [Withdraw Request, คำร้องถอนรายวิชา]
tags: [concept]
created: 2026-03-30
---

# Withdraw Request

## What

คำร้องขอถอนรายวิชา (Withdraw Request) คือหน่วยข้อมูลหลักของระบบ แต่ละคำร้องมี:
- **wid** (WITHDRAWID) — primary key ของ request record
- **studentcode** — รหัสนักศึกษา
- **classid / courseid** — รหัสกลุ่มเรียน / รหัสรายวิชา
- **acadyear / semester** — ปีการศึกษา / เทอม
- **reason** — เหตุผลที่ขอถอน
- **email / mobile** — ช่องทางแจ้งผล

**Status cycle:**

| สถานะ | ความหมาย |
|-------|----------|
| `null` / blank | คำร้องใหม่ รอพิจารณา |
| `W` | ขอเรียกพบ (Wait for Approve) |
| `A` | อนุมัติ (Approve) |
| `N` | ไม่อนุมัติ (Not Approve) |
| `C` | นักศึกษายกเลิกคำร้อง (Cancel) |

**กฎ Min-Course (Business Rule สำคัญ):**

```
get_num_course(studentcode) - 1 >= len(vdata)
```

นักศึกษาต้องเหลือรายวิชาอย่างน้อย 1 วิชาหลังถอน หากละเมิด API จะคืน HTTP 401:
```
"Student must study at least 1 course (can not withdrawn all courses)."
```

**Stored Procedures ที่เกี่ยวข้อง (`MIS_API_REG_W_STUDENT_PKG`):**

| Procedure | หน้าที่ |
|-----------|---------|
| `GET_COURSE_CAN_W` | นับจำนวนรายวิชาที่สามารถถอนได้ |
| `GET_STUDENT_COURSE` | ดึงรายวิชาที่ถอนได้ในเทอมนั้น |
| `ADD_REQUEST` | เพิ่มคำร้องถอน (คืน cursor + status + msg) |
| `GETREQUESTRESULT` | ผลการพิจารณาทุก wid |
| `GETREQUESTRESULTDETAIL` | รายละเอียดผล by wid |
| `GETREQUESTHISTORY` | ประวัติคำร้องทั้งหมด |
| `CANCELLIST` | รายการที่ยกเลิกได้ |
| `CANCELREQUEST` | ยกเลิกคำร้อง (ตั้ง status = `C`) |

## Why

การแยก record ต่อ wid ทำให้สามารถ track สถานะแต่ละวิชาอิสระกัน และรองรับ partial approval  
กฎ min-course ป้องกันนักศึกษาถอนทุกวิชาซึ่งจะเป็นปัญหาด้านสถานะการลงทะเบียน

## Related

- [[Withdraw Approval Workflow]] — กระบวนการอนุมัติ 3 ระดับ
- [[Withdraw Email Notification]] — การแจ้งผลให้นักศึกษาทาง email
- [[How to Submit Withdraw Request]] — คู่มือการยื่นคำร้อง
- [[ADR-001 Single Blueprint Multi-Role Path]] — การออกแบบ API path

Source: `misapi/reg/withdraw/routes.py`, `misapi/reg/withdraw/data_access.py`
