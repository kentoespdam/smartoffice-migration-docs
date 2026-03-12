# Mail Support — Category, Type, Statistic & System Notifications

> Module pendukung dan notifikasi otomatis.
> Source: `server/application/models/mailcategorymodel.php`, `mailtypemodel.php`, `mailstatisticreportmodel.php`, `mailmodel.php`

---

## Mail Category (Klasifikasi Surat) — CRUD Ringkasan

**Source:** `mailcategorymodel.php`
**Complexity:** Low (CRUD sederhana, tidak perlu diagram)

| Function | Description | Spring Boot |
|----------|-------------|-------------|
| `read()` | List kategori dengan paginasi + sort | `GET /api/mail-categories` |
| `save()` | Create/update kategori | `POST /api/mail-categories` |
| `del()` | Soft delete kategori | `DELETE /api/mail-categories/{id}` |

### Migration Notes
- Standard JPA CRUD. Gunakan `JpaRepository<MailCategory, Long>`.
- Field `sort` di-increment saat send() — pertahankan behavior ini.

---

## Mail Type (Jenis Surat) — CRUD Ringkasan

**Source:** `mailtypemodel.php`
**Complexity:** Low (CRUD sederhana, tidak perlu diagram)

| Function | Description | Spring Boot |
|----------|-------------|-------------|
| `read()` | List jenis surat dengan paginasi | `GET /api/mail-types` |
| `read_tree()` | Hierarchical tree view | `GET /api/mail-types/tree` |
| `save()` | Create/update jenis surat | `POST /api/mail-types` |
| `del()` | Soft delete | `DELETE /api/mail-types/{id}` |

### Migration Notes
- `read_tree()`: return tree structure. Bisa recursive CTE atau in-memory tree building.

---

## Mail Statistic Report — Ringkasan

**Source:** `mailstatisticreportmodel.php`
**Complexity:** Low (aggregation query)

| Function | Description | Spring Boot |
|----------|-------------|-------------|
| `getStatistic()` | Aggregation per period + category/org | `GET /api/reports/mail-statistics` |

### Migration Notes
- Native query atau `@Query` JPQL dengan GROUP BY.
- Data source: `mail_category_statistic` dan `mail_org_statistic` (di-maintain oleh send()).

---

## System Notification Diagrams

### create_publication_mail_notif() — Notifikasi Publikasi

**Source:** `mailmodel.php:337-442`
**Diagram type:** Sequence
**Complexity:** High

### What
Membuat surat notifikasi internal saat ada publikasi baru. Dikirim ke SEMUA karyawan aktif sebagai inbox. Pattern: resolve CLIENT_CODE config → generate nomor surat → load msg_template(2) → insert mail → loop all active users → insert recipient + inbox.

### Why
Publikasi (pengumuman perusahaan) harus sampai ke semua karyawan. Sistem otomatis membuat surat internal agar muncul di inbox setiap user.

### Diagram

```plantuml
@startuml
!theme plain
title create_publication_mail_notif() — Notifikasi Publikasi Baru\nmailmodel.php:337-442

participant "Caller\n(Publication Module)" as Caller
participant "MailModel" as Model
database "Database" as DB
participant "Kepegawaian API\n(target migrasi)" as API

Caller -> Model : create_publication_mail_notif(publication)

== Step 1: Resolve CLIENT_CODE Config ==

group CLIENT_CODE Branching
  alt BMS
    Model -> Model : m_type=1, m_category=86\nm_category_code="485.1"\nref_code="m_number_format_bms"
  else SMD
    Model -> Model : m_type=1, m_category=1\nm_category_code="000"\nref_code="m_number_format_smd"
  else BPN
    Model -> Model : m_type=1, m_category=3\nm_category_code="C"\nref_code="m_number_format_blp"
  end
  Model -> Model : user_id=2 (Administrator)
end

== Step 2: Generate Nomor Surat ==

Model -> DB : SELECT text FROM sys_reference\nWHERE code=ref_code
DB --> Model : no_template

Model -> Model : Replace placeholders:\n#org_code#→"00", #type#→"I"\n#MR#→bulan romawi, #YYYY#→tahun\n#m_cat#→m_category_code

Model -> DB : get_trans_sequence(\nTASK_MAIL, "00-"+m_category_code, 3)
DB --> Model : sequence number

Model -> Model : Replace #seq# → sequence\n→ kode surat final

== Step 3: Load & Fill Template ==

Model -> DB : SELECT message FROM msg_template\nWHERE template_id=2
DB --> Model : template

Model -> Model : Replace dalam template:\n#title#, #type#, #created_by_name#\n#created_by_title#, #published_date#

== Step 4: Insert Mail ==

Model -> DB : INSERT mail\n(m_no, m_date, m_type, m_category,\nm_subject="Publikasi baru",\nm_content, m_status=SENT,\nm_created_by=2/Administrator,\nm_to_str="Semua Karyawan")
DB --> Model : mail_id

Model -> DB : UPDATE mail SET\nm_root_id=mail_id, m_parent_id=0\nWHERE m_id=mail_id
note right
  Self-reference untuk
  threaded view (root of new thread)
end note

== Step 5: Create Sent Item (Admin) ==

Model -> DB : INSERT sys_user_task\n(user_id=2, tm_id=mail_id,\nfolder_id=SENT, read_status=READ)

== Step 6: Create Inbox untuk SEMUA Karyawan Aktif ==

Model -> DB : SELECT * FROM sys_user u\nJOIN employee e\nJOIN emp_profile p\nJOIN position ps\nWHERE user_status=1
note right #Orange
  **Migrasi:** Query ini JOIN tabel employee lokal.
  Di Spring Boot, ganti dengan:
  GET /pegawai?status=ACTIVE
  dari Kepegawaian API
end note
DB --> Model : all active users

loop untuk setiap user aktif
  Model -> DB : INSERT mail_recipient\n(mail_id, user_id, emp_id,\nemp_name, pos_id, pos_name,\ncirculation=4, email=0, sms=0)

  Model -> DB : INSERT sys_user_task\n(user_id, tm_id=mail_id,\nfolder_id=INBOX, read_status=UNREAD)
end

@enduml
```

### Migration Notes
- Broadcast ke semua user → potentially slow. Gunakan `@Async` + batch insert.
- Employee list → Kepegawaian API: `GET /pegawai?status=ACTIVE`
- Extract ke `SystemMailNotificationService` yang reusable.

---

### create_upd_emp_tanggungan_mail_notif() — Notifikasi Tanggungan

**Source:** `mailmodel.php:444-570`
**Diagram type:** Sequence
**Complexity:** Medium

### What
Notifikasi ke karyawan saat data tanggungan berubah (anak lepas tanggungan karena usia/pendidikan/nikah). Build detail HTML dari array lepas_tanggungan, template(4).

### Why
Perubahan tanggungan mempengaruhi tunjangan. Karyawan perlu tahu agar bisa verifikasi.

### Diagram

```plantuml
@startuml
!theme plain
title create_upd_emp_tanggungan_mail_notif() — Notifikasi Perubahan Tanggungan\nmailmodel.php:444-570

participant "Caller\n(Kepegawaian Module)" as Caller
participant "MailModel" as Model
database "Database" as DB

Caller -> Model : create_upd_emp_tanggungan_mail_notif(\nemp, lepas_tanggungan[])

== Step 1: Resolve CLIENT_CODE Config ==

group CLIENT_CODE Branching
  alt BMS
    Model -> Model : m_type=1, m_category=1 (UMUM)\nm_category_code="000"\nref_code="m_number_format_bms"
  else SMD
    Model -> Model : m_type=1, m_category=1\nm_category_code="000"\nref_code="m_number_format_smd"
  else BPN
    Model -> Model : m_type=1, m_category=1\nm_category_code="000"\nref_code="m_number_format_blp"
  end
  Model -> Model : user_id=2 (Administrator)
end

== Step 2: Generate Nomor Surat ==

Model -> DB : SELECT text FROM sys_reference\nWHERE code=ref_code
DB --> Model : no_template

Model -> Model : Replace placeholders\n(sama seperti notif lainnya)

Model -> DB : get_trans_sequence(\nTASK_MAIL, "00-000", 3)
DB --> Model : sequence

== Step 3: Build Detail HTML dari Tanggungan ==

Model -> Model : Load msg_template (template_id=4)

loop untuk setiap lepas_tanggungan
  Model -> Model : Build detail HTML:\n- Nama, Tgl Lahir, Usia\n- Status Pendidikan (0→"-", 1→"Belum Sekolah",\n  2→"Sekolah", 3→"Selesai Sekolah")\n- Status Nikah (0→"-", 1→"Belum Menikah",\n  2→"Telah Menikah")
end

Model -> Model : Replace #detail# dalam template

== Step 4: Insert Mail ==

Model -> DB : INSERT mail\n(m_subject="Perubahan data tanggungan",\nm_to_str=emp.emp_name,\nm_status=SENT, ...)
DB --> Model : mail_id

Model -> DB : UPDATE mail SET\nm_root_id=mail_id, m_parent_id=0

== Step 5: Create Sent Item (Admin) ==

Model -> DB : INSERT sys_user_task\n(user_id=2, folder_id=SENT)

== Step 6: Create Inbox untuk Karyawan Terkait ==

Model -> DB : SELECT * FROM sys_user u\nJOIN employee e JOIN emp_profile p\nJOIN position ps\nWHERE user_status=1\nAND e.emp_id=emp.emp_id
note right #Orange
  **Migrasi:** Single employee lookup.
  Di Spring Boot: GET /pegawai/{nipam}/nipam
end note
DB --> Model : target user(s)

loop untuk setiap user (biasanya 1)
  Model -> DB : INSERT mail_recipient\n(circulation=4, email=0, sms=0)
  Model -> DB : INSERT sys_user_task\n(folder_id=INBOX, read_status=UNREAD)
end

@enduml
```

### Migration Notes
- Detail HTML building → Thymeleaf template atau utility method
- Trigger dari modul Kepegawaian via Spring Event

---

### create_approval_cuti_notif() — Notifikasi Cuti

**Source:** `mailmodel.php:572-675`
**Diagram type:** Sequence
**Complexity:** Medium

### What
Notifikasi approval/rejection cuti ke karyawan. Subject dan template ID di-pass dari caller module. Paling fleksibel dari semua notif karena caller menentukan content.

### Why
Karyawan perlu notifikasi saat pengajuan cuti di-approve atau di-reject.

### Diagram

```plantuml
@startuml
!theme plain
title create_approval_cuti_notif() — Notifikasi Approval Cuti\nmailmodel.php:572-675

participant "Caller\n(Cuti Module)" as Caller
participant "MailModel" as Model
database "Database" as DB

Caller -> Model : create_approval_cuti_notif(\nemp, detail, subject, msg_tpl)
note right
  Berbeda dari notif lain:
  - subject dan msg_tpl di-pass dari caller
  - detail sudah berupa HTML string
end note

== Step 1: Resolve CLIENT_CODE Config ==

group CLIENT_CODE Branching
  alt BMS
    Model -> Model : m_type=1, m_category=1 (UMUM)\nm_category_code="000"\nref_code="m_number_format_bms"
  else SMD
    Model -> Model : idem, ref_code="m_number_format_smd"
  else BPN
    Model -> Model : idem, ref_code="m_number_format_blp"
  end
  Model -> Model : user_id=2 (Administrator)
end

== Step 2: Generate Nomor Surat ==

Model -> DB : SELECT text FROM sys_reference\nWHERE code=ref_code
DB --> Model : no_template
Model -> Model : Replace placeholders + get_trans_sequence
Model -> Model : → kode surat final

== Step 3: Load & Fill Template ==

Model -> DB : SELECT message FROM msg_template\nWHERE template_id=msg_tpl
note right
  msg_tpl di-pass dari caller,
  bukan hardcoded seperti notif lain
end note
DB --> Model : template
Model -> Model : Replace #detail# → detail HTML

== Step 4: Insert Mail ==

Model -> DB : INSERT mail\n(m_subject=subject (dari caller),\nm_to_str=emp.emp_name,\nm_status=SENT, ...)
DB --> Model : mail_id

Model -> DB : UPDATE mail SET\nm_root_id=mail_id, m_parent_id=0

== Step 5: Sent Item (Admin) ==

Model -> DB : INSERT sys_user_task\n(user_id=2, folder_id=SENT)

== Step 6: Inbox untuk Karyawan ==

Model -> DB : SELECT * FROM sys_user u\nJOIN employee e ...\nWHERE emp_id=emp.emp_id
DB --> Model : target user

loop untuk setiap user
  Model -> DB : INSERT mail_recipient\n(circulation=4)
  Model -> DB : INSERT sys_user_task\n(folder_id=INBOX, read_status=UNREAD)
end

@enduml
```

### Migration Notes
- Caller passing subject + template → generic notification pattern
- Bisa di-generalize menjadi method dengan template + context params

---

### create_archive_mail_notif() — Notifikasi Arsip

**Source:** `mailmodel.php:1617-1711`
**Diagram type:** Sequence
**Complexity:** Medium

### What
Notifikasi ke user yang diberi hak akses arsip baru. Template(3) dengan placeholder: ma_no, ma_subject, ma_archive_by_name, ma_secret_type. Recipient sudah resolved (user object dari get_allowed_user).

### Why
User yang diberi akses arsip perlu tahu agar bisa mengakses dokumen.

### Diagram

```plantuml
@startuml
!theme plain
title create_archive_mail_notif() — Notifikasi Hak Akses Arsip\nmailmodel.php:1617-1711

participant "Caller\n(Archive Module)" as Caller
participant "MailModel" as Model
database "Database" as DB

Caller -> Model : create_archive_mail_notif(user, archive)
note right
  Berbeda dari notif lain:
  - Recipient sudah resolved (user object)
  - Tidak perlu query employee
  - Template replace: archive fields
end note

== Step 1: Resolve CLIENT_CODE Config ==

group CLIENT_CODE Branching
  alt BMS
    Model -> Model : m_type=1, m_category=1 (UMUM)\nm_category_code="000"\nref_code="m_number_format_bms"
  else SMD
    Model -> Model : idem, ref_code="m_number_format_smd"
  else BPN
    Model -> Model : m_type=1, m_category=3 (PENGUMUMAN)\nm_category_code="C"\nref_code="m_number_format_blp"
    note right
      BPN berbeda: category=3,
      code="C" (bukan "000")
    end note
  end
  Model -> Model : user_id=2 (Administrator)
end

== Step 2: Generate Nomor Surat ==

Model -> DB : SELECT text FROM sys_reference\nWHERE code=ref_code
DB --> Model : no_template
Model -> Model : Replace placeholders\norg_code="00", type="I"
Model -> DB : get_trans_sequence(\nTASK_MAIL, "00-"+m_category_code, 3)
DB --> Model : sequence
Model -> Model : → kode surat final

== Step 3: Load & Fill Template ==

Model -> DB : SELECT message FROM msg_template\nWHERE template_id=3
DB --> Model : template

Model -> Model : Replace dalam template:\n#ma_no# → archive.ma_no\n#ma_subject# → archive.ma_subject\n#ma_archive_by_name# → archive.ma_archive_by_name\n#ma_secret_type# → archive.ma_secret_type

== Step 4: Insert Mail ==

Model -> DB : INSERT mail\n(m_subject="Pemberitahuan Hak Akses\nDokumen Arsip",\nm_to_str=user.emp_name + " (" +\nuser.pos_name + ")",\nm_status=SENT, ...)
DB --> Model : mail_id

Model -> DB : UPDATE mail SET\nm_root_id=mail_id, m_parent_id=0

== Step 5: Sent Item (Admin) ==

Model -> DB : INSERT sys_user_task\n(user_id=2, folder_id=SENT)

== Step 6: Inbox — Single Recipient ==

note over Model
  Tidak perlu query employee.
  User object sudah lengkap dari caller
  (via get_allowed_user di archive module)
end note

Model -> DB : INSERT mail_recipient\n(user_id, emp_id, emp_name,\npos_id, pos_name, circulation=4)

Model -> DB : INSERT sys_user_task\n(user_id, folder_id=INBOX,\nread_status=UNREAD)

@enduml
```

### Migration Notes
- Dipanggil oleh scheduled notification job (bukan realtime)
- User object dari Kepegawaian API (via get_allowed_user)

---

