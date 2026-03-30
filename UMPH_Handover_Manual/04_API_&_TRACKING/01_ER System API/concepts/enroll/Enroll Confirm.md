---
type: concept
system: reg-enroll
aliases: [Enroll Confirmation, ยืนยันลงทะเบียน, Confirm]
tags: [concept]
created: 2026-03-30
---

# Enroll Confirm

## What

การยืนยัน (Confirm) เป็นขั้นตอนสุดท้ายของแต่ละ phase เพื่อ "lock" รายการส่ง batch ไป Oracle — มี 2 จุดยืนยัน:

### Phase 1 Confirm — `POST /preenrollconfirm`
**Procedure:** `MIS_API_REG_ENROLL_PRE_ENROLL.PRE_REG_CONFIRM`

Parameters:
| Field | ความหมาย |
|-------|----------|
| `studentid` | รหัสนักศึกษา |
| `acadyear` | ปีการศึกษา |
| `semester` | เทอม |
| `schedule_period` | รอบของการแจ้งความประสงค์ |
| `user_type` | ประเภทผู้ใช้ |
| `method` | วิธีการยืนยัน |
| `submit_userid` | ผู้ยืนยัน |

คืน: `data` (cursor) + `status` + `message`

### Phase 2 Confirm — `POST /enrollconfirm`
**Procedure:** `MIS_API_REG_ENROLL_PKG.ENROLL_CONFIRM`

Parameters:
| Field | ความหมาย |
|-------|----------|
| `studentid`, `acadyear`, `semester` | ระบุเทอม |
| `user_type` | ประเภทผู้ใช้ |
| `submit_userid` | ผู้ยืนยัน |
| `submit_ip` | IP ผู้ยืนยัน (ดึงอัตโนมัติจาก `utils.getHost()`) |

คืน: `data` (cursor) + `status` + `message`

**Response pattern เหมือนกันทั้งสอง confirm:**
```python
result = utils.getStatusSuccessWithDataWithMessage(data, message, status)
```

**Validation pattern:**
```python
# user_type อนุญาต None (optional)
if not user_type and user_type != None:
    return 403 'Missing user_type parameter'
```

## Why

การมี confirm step บังคับให้นักศึกษาตรวจสอบรายการครั้งสุดท้ายก่อน commit ไป DB  
`submit_ip` ถูกดึงอัตโนมัติเหมือนกับ `create_ip` ใน Enrollment เพื่อป้องกัน IP spoofing  
`user_type` เป็น optional เพราะบางกรณีระบบ confirm แทนนักศึกษา (เช่น auto-confirm)

## Related

- [[Enroll Pre-Enrollment]] — รายการที่ confirm ใน phase 1
- [[Enrollment]] — รายการที่ confirm ใน phase 2
- [[How to Confirm Enrollment]] — คู่มือขั้นตอน confirm
- [[ADR-004 Two-Phase Enrollment Design]] — เหตุผลการมี 2 confirm steps

Source: `misapi/reg/enroll/routes.py`, `misapi/reg/enroll/data_access.py`
