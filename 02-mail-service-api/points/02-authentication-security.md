# 2. Authentication & Security

## 2.1 Token Flow

```
Client → AppWrite Login → JWT Token
Client → Mail Service API (Authorization: Bearer <token>)
Mail Service → AppWrite SDK → Validate Token → Extract User Context
```

## 2.2 Security Filter (Spring Boot)

Setiap request melewati `AppWriteAuthenticationFilter`:

1. Extract `Authorization: Bearer <token>` dari header
2. Call AppWrite Server SDK: `users.get(userId)` / validate session
3. Populate `SecurityContext` dengan `UserPrincipal`:

```json
{
  "userId": 123,
  "appwriteId": "abc123",
  "empId": 456,
  "empName": "John Doe",
  "posId": 789,
  "posName": "Kepala Bagian",
  "orgId": 10,
  "orgCode": "HRD",
  "mailCode": "05",
  "officeCode": "PUSAT",
  "roles": ["ROLE_USER", "ROLE_MAIL_ADMIN"]
}
```

> **Note**: Mapping `appwriteId` → `userId/empId` dilakukan via tabel bridging `sys_user` pada fase transisi, kemudian dimigrasi ke AppWrite user metadata.

## 2.3 Authorization Matrix

| Resource | USER | MAIL_ADMIN | SUPER_ADMIN |
|---|---|---|---|
| Own mail & folders | ✅ CRUD | ✅ CRUD | ✅ CRUD |
| Other user's mail | ❌ | Read only (report) | ✅ Full |
| Mail Types & Categories | Read | ✅ CRUD | ✅ CRUD |
| Archive (own org) | ✅ CRUD | ✅ CRUD | ✅ CRUD |
| Archive access control | ❌ | ✅ CRUD | ✅ CRUD |
| SMTP / Notif config | ❌ | ❌ | ✅ CRUD |
| Reports | Own data | All org | All org |

<!-- nav -->
---
**Navigation:** [< Prev](01-overview.md) | [Next >](03-common-conventions.md)
<!-- /nav -->
