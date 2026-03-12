# 🔍 Non-Trivial Business Logic - Modul Persuratan

## Quick Navigation

- [1. send() - Pengiriman Surat](#sec-send)
- [2. generate_code() - Auto Numbering Surat](#sec-generate-code)
- [3. make_nlevel_threaded() + find_node()](#sec-thread-tree)
- [4. directArsip() - Direct Archive](#sec-direct-arsip)
- [5. signMe() + checkSign()](#sec-sign)
- [6. create_*_notif() - System Notifications](#sec-system-notif)
- [7. Archive generate_code()](#sec-archive-generate-code)
- [8. Archive Access Control](#sec-archive-access)
- [Complexity Summary](#sec-complexity-summary)

---

<a id="sec-send"></a>
### 1. 📧 send() — Pengiriman Surat (Paling Kompleks)

File: `mailmodel.php:812-1018`

Ini bukan sekadar update status. Ada 10 side-effects dalam satu transaksi:

| Step | Logic | Spring Boot Equivalent |
|---|---|---|
| 1 | Validasi recipient, gagal jika belum ada penerima | `@PreCondition` / service validation |
| 2 | Auto-generate nomor surat via `generate_code()` | `MailCodeGeneratorService` |
| 3 | Update status ke `MAIL_STATUS_SENT` + set `m_created_date` | JPA update |
| 4 | Email notification queue: cek `smtp_mail_config`, isi template dari `msg_template`, masukkan ke `smtp_mail_log` | `MailNotificationService` + `@Async` |
| 5 | Create inbox untuk setiap recipient di `sys_user_task` (`folder=INBOX`, `status=UNREAD`) | Event-driven / batch insert |
| 6 | Update statistik `mail_category_statistic` dan `mail_org_statistic` (monthly aggregation, upsert) | `MailStatisticService` |
| 7 | Move sender draft ke sent di `sys_user_task` | JPA update |
| 8 | Mark parent mail as READ jika ini reply (`m_parent_id != 0`) | JPA update |
| 9 | Response time tracking: jika reply, hitung `TIMEDIFF` dan insert ke `mail_respontime` | `MailResponseTimeService` |
| 10 | Trigger SMTP send jika `notif_method == 1` (realtime) | `@Async` SMTP sender |

> ⚠️ Catatan migrasi: Ini harus di-refactor jadi transactional orchestration dengan `@Transactional` + event listeners untuk decoupling (notif, statistik, response time).

---

<a id="sec-generate-code"></a>
### 2. 🔢 generate_code() — Auto Numbering Surat

File: `mailmodel.php:1564-1617`

Logic: Template-based nomor surat dengan multi-tenant client logic:

```text
Template: #seq#/1421002/#org_code#-#type#/#MR#/#YYYY#-#m_cat#
Output:   001/1421002/01-I/III/2026-000
```

| Placeholder | Source |
|---|---|
| `#org_code#` | User's `mail_code` (dari organization) |
| `#type#` | `I=Internal`, `M=Memo`, `E/K=Eksternal` (varies per `CLIENT_CODE`) |
| `#MR#` | Bulan Romawi (helper `get_month_r`) |
| `#YYYY#` | Tahun |
| `#m_cat#` | Kode klasifikasi surat |
| `#seq#` | Auto-increment per prefix via `get_trans_sequence()` |

> ⚠️ Multi-client branching: `CLIENT_CODE` (`BMS`/`SMD`/`BPN`) punya template dan type mapping berbeda. Perlu di-refactor jadi Strategy Pattern di Spring Boot.

---

<a id="sec-thread-tree"></a>
### 3. 🌳 make_nlevel_threaded() + find_node() — Thread Tree Builder

File: `direct/mail.php:219-286`

Membangun n-level nested tree dari flat mail data untuk ExtJS TreePanel:

- Setiap mail punya `m_root_id` (root thread) dan `m_parent_id` (direct parent)
- Rekursif `find_node()` mencari parent node dalam tree dan menempel child
- Fallback: jika parent tidak ditemukan, taruh di level 1 child dari root
- Fallback 2: jika root juga tidak ditemukan, taruh di top level

> Spring Boot: Bisa pakai recursive CTE query atau tetap in-memory tree building. Cocok untuk `MailThreadService.buildTree()`.

---

<a id="sec-direct-arsip"></a>
### 4. 📁 directArsip() — Direct Archive dari Surat

File: `direct/mail.php:364-395`

Logic flow:

1. Cek role permission — user harus punya akses menu arsip (sys_role_menu_event)
2. Cek lampiran — surat HARUS punya attachment, jika tidak → reject
3. Cek duplikat — cek m_ma_id di root mail, jika sudah diarsip → reject dengan info siapa yang arsip
4. Copy attachments — dari REF_TYPE_MAIL ke REF_TYPE_MAIL_ARCHIVE dengan temp_id

> Spring Boot: Ini jadi workflow validation chain. Cocok untuk `ArchiveService` dengan `@PreAuthorize` + custom validators.

---

<a id="sec-sign"></a>
### 5. ✍️ signMe() + checkSign() — Digital Signature

File: `direct/mail.php:395-420`

Logic:

- `signMe()`: Generate `uniqid()`, simpan ke `print_log` (`auth_code`, date, `mail_id`, username, IP)
- `checkSign()`: Lookup `print_log` by `auth_code`, render view `cek_signature.html`

> Spring Boot: Ini bukan digital signature sebenarnya, lebih ke print verification code. Bisa di-improve dengan JWT/UUID + QR code. Map ke `GET /api/mails/verify-sign/{key}`.

---

<a id="sec-system-notif"></a>
### 6. 📮 create_*_notif() — System-Generated Mail Notifications (3 variants)

File: `mailmodel.php:337-750`

Tiga jenis notifikasi otomatis yang membuat surat internal (bukan email):

| Function | Trigger | Recipient |
|---|---|---|
| `create_publication_mail_notif` | Publikasi baru dibuat | Semua karyawan aktif |
| `create_upd_emp_tanggungan_mail_notif` | Data tanggungan berubah | Karyawan terkait |
| `create_approval_cuti_notif` | Approval cuti | Karyawan terkait |
| `create_archive_mail_notif` | Arsip baru + hak akses | User yang diberi akses |

Semua mengikuti pola yang sama:

1. Generate nomor surat (per CLIENT_CODE)
2. Isi template dari msg_template
3. Insert ke mail table
4. Set m_root_id = self (new thread)
5. Create sys_user_task (sent untuk admin, inbox untuk recipients)

> Spring Boot: Extract ke SystemMailService yang dipanggil dari modul lain via Spring Events (@EventListener). Eliminasi duplikasi code.

---

<a id="sec-archive-generate-code"></a>
### 7. 🗄️ Archive generate_code() — Nomor Arsip (Terpisah dari Nomor Surat)

File: `mailarchivemodel.php` (bottom)

Sama kompleksnya dengan mail generate_code() tapi dengan:
- Office-based branching (PUSAT vs cabang → prefix berbeda)
- Sequence per office → prefix_sequence = office_code + type + year
- Template berbeda per CLIENT_CODE (BMS/SMD/BPN)

---

<a id="sec-archive-access"></a>
### 8. 🔐 Archive Access Control

File: `mailarchivemodel.php`

- `set_access()`: Delete-and-reinsert pattern untuk permission per position
- `search()`: Join `mail_archive_access` untuk filter berdasarkan `pos_id` user + access flag
- 3 permission levels: access (read), download, history
- Archive notification workflow: `set_notif()` -> `get_unnotified()` -> `get_allowed_user()` -> `cek_user_notif()` -> `set_user_notif()`

> Spring Boot: Map ke `ArchiveAccessService` + `@PreAuthorize` custom. Notification bisa jadi scheduled job (`@Scheduled`).

---

<a id="sec-complexity-summary"></a>
### 📊 Complexity Summary
| Function | Complexity | Refactor Priority | Notes |
|---|---|---|---|
| `send()` | 🔴 Very High | Must refactor | 10 side-effects, needs event-driven |
| `generate_code()` (mail) | 🟠 High | Strategy Pattern | Multi-tenant branching |
| `generate_code()` (archive) | 🟠 High | Strategy Pattern | Office + client branching |
| `make_nlevel_threaded()` | 🟡 Medium | Keep or use CTE | Tree building logic |
| `directArsip()` | 🟡 Medium | Validation chain | Permission + business rules |
| `create_*_notif()` x4 | 🟠 High | DRY via template | 80% duplicated code |
| `signMe/checkSign` | 🟢 Low | Enhance security | Currently just `uniqid` |
| Archive access control | 🟡 Medium | Spring Security | Position-based ACL |