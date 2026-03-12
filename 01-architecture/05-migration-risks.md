# 5. KEY MIGRATION RISKS & RECOMMENDATIONS

[← Kembali ke README](./README.md) | [← UML Visualization](./04-uml-visualization.md)

---

## 5.1 Data Volume Consideration

| Table | Records (est.) | Impact |
|-------|---------------|--------|
| `mail` | ~1.8M | Paginasi wajib, index on m_root_id, m_created_date |
| `mail_recipient` | ~2.2M | Index composite (mail_id, user_id) |
| `attachments` | ~5M | Composite PK (id, upload_date) — pertahankan untuk partitioning |
| `sys_user_task` | ~4-5M (estimated) | Index composite (user_id, folder_id) — critical for inbox query |

## 5.2 Multi-Tenant Logic

Legacy system memiliki hardcoded `CLIENT_CODE` (BMS, SMD, BPN) untuk:
- Format nomor surat berbeda per-tenant
- Sort order inbox berbeda per-tenant
- Logika arsip berbeda per-tenant

**Recommendation:** Extract ke Strategy Pattern + configuration table, bukan hardcode.

## 5.3 `sys_user_task` Migration Decision

Tabel ini **bukan bagian dari `sys_` namespace** secara logis. Rekomendasi: rename menjadi `mail_user_task` atau `mail_routing` di service baru untuk kejelasan ownership.

## 5.4 Attachment Physical Storage

Legacy menggunakan filesystem lokal (`/attachments/`). Untuk microservice:
- **Phase 1:** Tetap filesystem dengan shared mount (NFS/EFS)
- **Phase 2:** Migrasi ke object storage (MinIO/S3) untuk scalability

---

*Document generated from analysis of SmartOffice legacy codebase*
*Schema: smartoffice.sql (MariaDB 11.1.5)*
*Source: PHP CodeIgniter 2 RPC Model → Target: Spring Boot REST API*
