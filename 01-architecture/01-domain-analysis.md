# 1. DOMAIN ANALYSIS (Analisis Domain)

[← Kembali ke README](./README.md)

---

## 1.1 Entitas Inti yang Teridentifikasi dari Legacy System

Berdasarkan analisis schema database (`smartoffice.sql`) dan model PHP (`mailmodel.php`, `mailarchivemodel.php`, `mailcategorymodel.php`, `mailtypemodel.php`, `attachmentmodel.php`), berikut adalah entitas-entitas inti dalam domain Persuratan:

### Core Entities (Entitas Utama)

| Entitas | Tabel Legacy | Deskripsi | AUTO_INCREMENT |
|---------|-------------|-----------|----------------|
| **Mail** | `mail` | Surat utama (internal/external, masuk/keluar) | 1,804,865 |
| **MailRecipient** | `mail_recipient` | Penerima surat dengan sirkulasi | 2,250,600 |
| **MailFolder** | `mail_folder` | Folder organisasi surat per-user | 1,840 |
| **MailArchive** | `mail_archive` | Arsip surat dengan lokasi fisik | 39,893 |
| **MailArchiveAccess** | `mail_archive_access` | Kontrol akses arsip per-jabatan | 167,470 |
| **Attachment** | `attachments` | Lampiran file (polymorphic: Mail & Arsip) | 5,088,441 |

### Master Data Entities (Data Referensi)

| Entitas | Tabel Legacy | Deskripsi |
|---------|-------------|-----------|
| **MailType** | `mail_type` | Jenis surat (Internal, Memo, External) |
| **MailCategory** | `mail_category` | Kategori/klasifikasi surat |

### Supporting Entities (Entitas Pendukung)

| Entitas | Tabel Legacy | Deskripsi |
|---------|-------------|-----------|
| **MailResponseTime** | `mail_respontime` | Tracking waktu respons surat |
| **MailCategoryStatistic** | `mail_category_statistic` | Statistik per-kategori per-bulan |
| **MailOrgStatistic** | `mail_org_statistic` | Statistik per-organisasi per-bulan |
| **MailArchiveNotif** | `mail_archive_notif` | Flag notifikasi arsip baru |
| **MailArchiveNotifLog** | `mail_archive_notif_log` | Log notifikasi arsip per-user |
| **AttachmentDownloadHistory** | `attachment_download_history` | Histori download lampiran |
| **SmtpMailConfig** | `smtp_mail_config` | Konfigurasi SMTP |
| **SmtpMailLog** | `smtp_mail_log` | Log pengiriman email SMTP |

## 1.2 Business Logic yang Teridentifikasi

Dari analisis `mailmodel.php` (1000+ baris), berikut proses bisnis utama:

### A. Mail Lifecycle (Siklus Hidup Surat)
```
Draft → Sent → Inbox (Recipient) → Read → [Reply/Forward] → Archived/Deleted
```

**Status Flow:**
- `MAIL_STATUS_DRAFT` — Surat dibuat, belum dikirim
- `MAIL_STATUS_SENT` — Surat sudah terkirim ke semua penerima
- Folder-based routing via `sys_user_task`: INBOX, SENT, DRAFT, READ, DELETED, PERSONAL(>10)

### B. Mail Number Generation (Penomoran Surat)
- **Pola:** `#seq#/#org_code#-#type#/#MR#/#YYYY#-#m_cat#`
- **Multi-tenant:** Logika berbeda per `CLIENT_CODE` (BMS, SMD, BPN)
- **Sequence:** Menggunakan `get_trans_sequence()` dengan prefix unik
- **Why penting:** Penomoran surat resmi adalah business-critical, harus atomic dan unique

### C. Threaded Mail (Surat Berantai)
- `m_root_id` — ID surat asal (root thread)
- `m_parent_id` — ID surat parent langsung
- Mendukung n-level threading untuk disposisi/balasan berantai
- Reply ke surat memindahkan parent dari INBOX ke READ

### D. Recipient & Circulation (Penerima & Sirkulasi)
- Setiap surat memiliki multiple recipients
- **Circulation types:** Tembusan, Disposisi, dll (via `sys_reference.code='sirkulasi'`)
- Recipient di-denormalisasi (`emp_name`, `pos_name`) untuk performance
- Flag `email` dan `sms` per-recipient untuk notifikasi

### E. Archive Workflow
```
Mail → DirectArsip (check attachment) → Create MailArchive → Set Access → Set Notif → Published
```
- **Access Control:** Per-jabatan (pos_id), bukan per-user
- **Physical Location:** Building → Floor → Room → Rack → Tier → Box
- **Secret Type:** Tingkat kerahasiaan dokumen
- **Keyword Indexing:** Full-text search via keyword field

### F. Notification Pipeline
```
Send Mail → smtp_mail_log (queue) → SmtpModel.send() → Email delivery
                                  → Firebase push notification
```

## 1.3 Tight Coupling dengan HR (Kepegawaian) yang Harus Diputus

Dari kode legacy, berikut JOIN langsung ke tabel HR yang **harus diubah** menjadi API call:

| Lokasi | Tabel HR yang Di-JOIN | Data yang Diambil | Alternatif REST |
|--------|----------------------|-------------------|-----------------|
| `mailmodel.php:read_folder` | `sys_user`, `employee`, `emp_profile` | user_emp_id, emp_name, emp_profile_id | GET `/api/employees/{empId}` |
| `mailmodel.php:send` | `employee`, `emp_profile`, `position` | emp_email, pos_name, pos_org_id | GET `/api/employees/{empId}/position` |
| `mailmodel.php:save_recipient` | `employee`, `emp_profile`, `position` | emp_name, pos_id, pos_name | GET `/api/employees/{empId}` |
| `mailarchivemodel.php:get_allowed_user` | `employee`, `emp_profile`, `sys_user`, `position` | user_id, emp_name, pos_name | GET `/api/positions/{posId}/employees` |
| `mailmodel.php:create_publication_notif` | `sys_user`, `employee`, `emp_profile`, `position` | All active users | GET `/api/employees?status=active` |

---

[Selanjutnya: Decomposition Strategy →](./02-decomposition-strategy.md)
