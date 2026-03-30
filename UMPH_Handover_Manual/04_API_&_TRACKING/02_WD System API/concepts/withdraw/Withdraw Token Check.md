---
type: concept
system: reg-withdraw
aliases: [Token Check, JWT Conversion, tokencheck]
tags: [concept]
created: 2026-03-30
---

# Withdraw Token Check

## What

Endpoint `GET /tokencheck` ทำหน้าที่แปลง **External JWT** (จาก Portal อื่น) → **MISAPI JWT** เพื่อให้ใช้งาน API ของ MISAPI ได้

**Flow:**

```
1. Client ส่ง External JWT ใน header "token"
2. ถอดรหัส JWT ด้วย secret: "lasdkjliwfekflasifjaaaa"
3. ดึงข้อมูลจาก user_claims.data:
   - USERID, USERNAME, USERTYPE, OFFICERID
4. สร้าง UserObject และ MISAPI JWT ใหม่ ด้วย create_access_token()
5. ส่งกลับ { new_token: { token, status } }
```

**Request:**
```http
GET /reg/withdraw/tokencheck
token: <external_jwt>
```

**Response (200):**
```json
{
  "message": "JWT is valid!",
  "decoded_token": { "data": { "USERID": "...", ... } },
  "new_token": { "token": "<misapi_jwt>", "status": "success" }
}
```

**Error responses:**
- `401` — `"Signature has expired."` (expired token)
- `401` — `"Invalid token."` (invalid signature/format)

**External JWT structure expected:**
```json
{
  "user_claims": {
    "data": {
      "USERID": "...",
      "USERNAME": "...",
      "USERTYPE": "...",
      "OFFICERID": "..."
    }
  }
}
```

> [!warning] Security Note
> - Secret key `"lasdkjliwfekflasifjaaaa"` hardcoded ใน routes.py — ควรย้ายไป environment variable
> - `@jwt_required` และ `@authen_roles` ถูก comment out — endpoint นี้ **ไม่มี authentication**

## Why

Endpoint นี้รองรับ Single Sign-On จาก Student Portal ที่ใช้ JWT ต่าง secret  
แทนที่จะ implement OAuth2 / OIDC ใหม่ทั้งหมด ใช้วิธี token exchange อย่างง่ายเพื่อ bridge ระหว่าง 2 ระบบ

## Related

- [[Withdraw Approval Workflow]] — ต้องใช้ MISAPI JWT จาก tokencheck เพื่อเรียก protected endpoints
- [[ADR-001 Single Blueprint Multi-Role Path]] — context ของ Blueprint design

Source: `misapi/reg/withdraw/routes.py`
