---
type: decision
system: reg-withdraw
date: 2026-03-30
status: accepted
tags: [decision]
---

# ADR-002 Oracle Function vs Procedure

## Context

Oracle DB ของระบบมีทั้ง Stored Procedures และ Stored Functions ในบาง Package  
`MIS_API_REG_W_MASTER_PKG.FUNC_GET_ACADYEAR` เป็น Oracle Function ที่ return ค่า scalar (ปีการศึกษาปัจจุบัน) โดยตรง ไม่ใช่ Stored Procedure ที่ใช้ OUT parameter

ต้องตัดสินใจวิธีเรียกจาก Python (cx_Oracle)

## Decision

ใช้ `OracleDb.call_func()` สำหรับ Oracle Function ที่ return scalar value และใช้ `call_proc()` / `call_proc_cursor()` สำหรับ Stored Procedures ทั่วไป:

```python
# Oracle Function → call_func
def get_acadyear_semester():
    dbase = OracleDb()
    proc = 'MIS_API_REG_W_MASTER_PKG.FUNC_GET_ACADYEAR'
    result = dbase.call_func(proc)   # คืนค่า scalar โดยตรง
    return result

# Stored Procedure → call_proc
def get_num_course(vuserid):
    dbase = OracleDb()
    proc = 'MIS_API_REG_W_STUDENT_PKG.GET_COURSE_CAN_W'
    cur = dbase.cursor
    vcount = cur.var(cx_Oracle.STRING)
    params = [vuserid, vcount]
    result = dbase.call_proc(proc, params=params)
    return [vcount.getvalue(0)]

# Stored Procedure with implicit cursor → call_proc_cursor
def GetFaculty(vfacultyid, vlang):
    dbase = OracleDb()
    proc = 'MIS_API_REG_MASTER_PKG.GetFaculty'
    result = dbase.call_proc_cursor(proc, params=[vfacultyid, vlang])
    return result
```

## Rationale

cx_Oracle แยกการเรียก Function และ Procedure — การใช้ `call_func` กับ Oracle Function ให้ผลที่ถูกต้องและ clean กว่าการ wrap function call ใน SQL

| Pattern | ใช้เมื่อ |
|---------|---------|
| `call_func(proc)` | Oracle Function — return scalar ไม่มี OUT param |
| `call_proc(proc, params)` | Procedure ที่มี OUT parameter (OUT var สร้างด้วย `cur.var()`) |
| `call_proc_cursor(proc, params)` | Procedure ที่ return implicit REF CURSOR |

## Related

- [[Withdraw Calendar]] — ใช้ `call_func` สำหรับ `FUNC_GET_ACADYEAR`
- [[How to Manage Withdraw Calendar]] — ตัวอย่างการใช้ acadyear ที่ได้

Source: `misapi/reg/withdraw/data_access.py`
