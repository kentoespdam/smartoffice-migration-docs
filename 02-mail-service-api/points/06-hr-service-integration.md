# 6. HR Service Integration Points

Mail Service mengonsumsi data personil dari HR Service (`http://192.168.1.214:8080`).

## 6.1 Data yang Dikonsumsi

| Data | Dipakai Untuk | HR Endpoint (Expected) |
|---|---|---|
| Employee (empId, empName, empCode) | Recipient lookup, creator info | `GET /v1/employees/{empId}` |
| Employee email | Email notification | `GET /v1/employees/{empId}` → `.email` |
| Position (posId, posName) | Recipient jabatan, archive access | `GET /v1/positions/{posId}` |
| Organization (orgId, orgName, mailCode) | Nomor surat, statistik, arsip | `GET /v1/organizations/{orgId}` |
| Employee by Position | Archive notification recipients | `GET /v1/positions/{posId}/employees` |
| Employee Search | Recipient picker di frontend | `GET /v1/employees?keyword=...` |
| Location (building/floor/room) | Arsip lokasi fisik | `GET /v1/locations?type=...` |
| User → Employee mapping | Auth context building | `GET /v1/users/{userId}/employee` |

## 6.2 Caching Strategy

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────┐
│  Mail API   │────▶│  Local Cache     │────▶│  HR Service  │
│  Request    │     │  (Redis, 5min)   │     │  REST API    │
└─────────────┘     └──────────────────┘     └──────────────┘
```

- **Read-through cache**: Pertama cek Redis, jika miss → call HR Service → simpan di Redis
- **TTL**: 5 menit untuk employee/position data (bisa berubah karena mutasi)
- **Denormalized copy**: Saat mail dikirim, `emp_name` dan `pos_name` di-snapshot ke `mail_recipient` dan `mail.m_created_by_name` (sama seperti legacy) untuk historical accuracy
- **Fallback**: Jika HR Service down, gunakan data denormalized terakhir di database

## 6.3 Circuit Breaker

Implementasi Resilience4j circuit breaker untuk HR Service calls:
- Failure rate threshold: 50%
- Wait duration in open state: 30 seconds
- Fallback: return cached/denormalized data

<!-- nav -->
---
**Navigation:** [< Prev](04-entity-relationship-mapping.md) | [Next >](07-async-processing-events.md)
<!-- /nav -->
