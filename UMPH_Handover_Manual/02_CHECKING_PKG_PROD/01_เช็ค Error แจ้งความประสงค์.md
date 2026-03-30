### **ตรวจสอบ Error การแจ้งความประสงค์ (Pre-register Error Check)**

**วัตถุประสงค์:** เพื่อใช้ตรวจสอบหาสาเหตุเมื่อนักศึกษาพบปัญหา (Error) ระหว่างการแจ้งความประสงค์ลงทะเบียนเรียน โดยเช็คตั้งแต่การตั้งค่าปลดล็อค (Abort Config), ข้อมูลรายวิชาที่เปิดสอน ไปจนถึงประวัติการทำรายการของนักศึกษาคนนั้นๆ **ผู้ร้องขอ:** เจ้าหน้าที่ทะเบียน หรือ Helpdesk ที่รับเรื่องจากนักศึกษา **ตารางที่เกี่ยวข้อง (Tables):** `avsreg.error_abort_config`, `avsreg.class`, `avsreg.course`, `avsreg.enrollsummary`

**SQL Scripts & Explanations:**

**1. เช็คตารางตั้งค่าการข้ามเงื่อนไข (Error Abort Config)** ใช้สำหรับดูเงื่อนไขการปลดล็อค Error ทั้งระบบ (ดึงข้อมูลภาพรวม)

SQL

```
SELECT * FROM avsreg.error_abort_config c;
```

**2. เช็คการปลดล็อค Error เป็นรายบุคคล (Specific Student Config)** ใช้ตรวจสอบว่านักศึกษารายนี้ (เช่น 6631210085 หรือ 6631006002) มีการตั้งค่า config ยกเว้น error ไว้หรือไม่ _💡 Pro-Tip: การ Select `t.rowid` พ่วงมาด้วย จะทำให้เราสามารถกด Edit เพื่อแก้ไขหรือลบข้อมูลบรรทัดนั้นผ่านหน้า Grid ของ PL/SQL Developer ได้ทันที_

SQL

```
SELECT t.*, t.rowid 
FROM avsreg.error_abort_config t 
WHERE t.studentid = 6631210085; -- เปลี่ยนรหัสนักศึกษาตามที่ได้รับแจ้ง
```

**3. เช็คข้อมูลการเปิดสอนของรายวิชาที่มีปัญหา (Class & Course Lookup)** ใช้ตรวจสอบว่ารายวิชาที่นักศึกษาพยายามจะลง (เช่น รหัสวิชา '2303222') ในปีการศึกษา 2568 เทอม 1 มีการเปิดสอนจริงๆ หรือไม่ และมีสถานะ Class เป็นอย่างไร

SQL

```
SELECT * FROM avsreg.class c  
INNER JOIN avsreg.course co ON co.courseid = c.courseid
WHERE c.acadyear = 2568 
  AND c.semester = 1 
  AND co.coursecode IN ('2303222'); -- เปลี่ยนรหัสวิชา และเทอมตามที่ต้องการเช็ค
```

**4. เช็คประวัติการลงทะเบียน/แจ้งความประสงค์ของนักศึกษา (Enrollment Summary)** ตรวจสอบดูว่าในเทอมนั้น (เช่น ปี 2568) นักศึกษาคนนี้ทำรายการวิชาอะไรไปแล้วบ้าง สถานะเป็นอย่างไร เพื่อหาความขัดแย้ง (Conflict) ที่อาจทำให้เกิด Error

SQL

```
SELECT co.coursecode, h.* FROM avsreg.enrollsummary h
INNER JOIN avsreg.course co ON co.courseid = h.courseid 
WHERE h.acadyear = 2568 
AND h.studentid = 6631006002; -- เปลี่ยนรหัสนักศึกษา และปีการศึกษาตามจริง
```

#ErrorCheck #PreRegister


