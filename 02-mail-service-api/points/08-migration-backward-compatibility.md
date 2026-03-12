# 8. Migration & Backward Compatibility

## 8.1 Phased Migration Plan

```
Phase 1: API Gateway + Spring Boot Mail Service
├── Frontend ExtJS → API Gateway → Spring Boot Mail Service
├── Database: shared MariaDB (smartoffice schema)
├── sys_user_task: tetap dipakai (no schema change)
├── Auth: AppWrite + sys_user bridging table
└── HR data: read from existing tables (employee, position, organization)

Phase 2: Decoupled HR Service
├── HR data → consumed via REST API from HR Service
├── Local cache (Redis) + denormalized snapshots
└── Remove direct dependency to employee/position/organization tables

Phase 3: Dedicated Database
├── Mail tables extracted to dedicated schema/database
├── Event-driven sync for cross-service data
└── Full microservice independence
```

## 8.2 Backward Compatibility Guarantees

| Legacy Concern | New API Approach |
|---|---|
| `sys_user_task` folder model | Tetap dipakai. API abstraksi di atasnya |
| `m_to_str` denormalized field | Tetap di-maintain saat create/update untuk backward compat |
| `CLIENT_CODE` branching (BMS/SMD/BPN) | Configurable number format via `sys_reference`, no hardcoded branching |
| ExtJS `build_where_condition` filter | API menggunakan explicit query params, server-side mapping ke DB query |
| Temp ID pattern (create → update ref) | Server-side: single transaction, no temp ID exposed to client |
| `folder_id = -1` permanent delete | Tetap dipakai. API menyembunyikan detail ini |

## 8.3 Number Format Configuration

Nomor surat digenerate dari template di `sys_reference`:

```
Template: #seq#/#org_code#-#type#/#MR#/#YYYY#-#m_cat#

Variables:
  #seq#       → Auto-increment sequence (padded, per prefix)
  #org_code#  → Kode organisasi user (dari HR Service)
  #type#      → I/M/E/K (internal/masuk/eksternal/keluar)
  #MR#        → Bulan romawi (I, II, III, ...)
  #YYYY#      → Tahun 4 digit
  #m_cat#     → Kode kategori surat

Sequence prefix: "{org_code}-{category_code}-{year}"
```

> **Improvement**: Pindahkan konfigurasi nomor surat ke tabel khusus `mail_number_format` yang configurable per tenant/office, menggantikan hardcoded `CLIENT_CODE` logic.

<!-- nav -->
---
**Navigation:** [< Prev](07-async-processing-events.md) | [Next >](../endpoints/5.1-folders.md)
<!-- /nav -->
