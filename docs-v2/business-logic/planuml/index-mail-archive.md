# Mail Archive — Business Logic Diagrams

> Modul arsip surat digital.
> Source: `server/application/direct/mailarchive.php` + `server/application/models/mailarchivemodel.php`

---

## archive() + create() + update() — Archive Workflow

**Source:** `mailarchivemodel.php:263-399`
**Diagram type:** Activity
**Complexity:** High

### What
Workflow arsip surat dengan 4 entry points: create (arsip baru dengan auto-numbering dan validasi organisasi), update (edit arsip existing dengan handling perubahan referensi surat), archive/finalize (set status=2), delete (soft delete status=3 dengan release referensi surat).

### Why
Arsip surat adalah proses yang melibatkan nomor arsip unik, referensi ke surat asli (m_ma_id), dan lokasi fisik penyimpanan. Referensi surat harus dikelola (book/release) agar satu surat hanya bisa di-arsip sekali.

### Diagram

```plantuml
@startuml
!theme plain
title archive() + create() + update() — Archive Workflow\nmailarchivemodel.php:263-399

start

note right
  Entry points:
  - Controller save() → create() atau update()
  - Controller archive() → finalize archive
  - Controller del() → soft delete
end note

if (Action?) then (save - baru)

  :create() — mailarchivemodel.php:263-319;

  partition "Validasi Organisasi" #LightBlue {
    :Query mail_code FROM organization
    WHERE org_id = ma_org_id;

    if (organization tidak ditemukan?) then (yes)
      #Pink:Return ERROR:
      "Unit Kerja tidak dikenal";
      stop
    else (no)
    endif
  }

  partition "Resolve Category Code" #LightBlue {
    if (ma_mcat_code kosong?) then (yes)
      :Query mcat_code FROM mail_category
      WHERE mcat_id = ma_mcat_id;
    else (no)
    endif
  }

  :generate_code(input);
  note right
    Lihat diagram:
    **archive-generate-code.puml**
  end note

  partition "Insert Archive" #LightBlue {
    :INSERT mail_archive:
    - ma_no = generated code
    - ma_mail_date, ma_mcat_type, ma_mcat_code
    - ma_mcat_id, ma_org_code, ma_org_id
    - ma_ref_id, ma_ref_no
    - ma_sent_to, ma_subject, ma_content, ma_note
    - ma_status (dari input)
    - ma_secret_type
    - ma_loc_* (building, floor, room, rack, tier, box)
    - ma_keyword, ma_keyword_index_flag=0
    - ma_archive_date=now()
    - ma_archive_by_name=user
    - office_code;
    :id = insert_id();
  }

  partition "Update Mail Reference" #LightBlue {
    if (ma_ref_id != 0?) then (yes)
      :UPDATE mail SET m_ma_id = archive_id
      WHERE m_id = ma_ref_id;
      note right
        Link surat ke arsip.
        Surat yang sudah di-link
        tidak bisa di-arsip lagi.
      end note
    else (no)
    endif
  }

  :Return success + id + code;

elseif (save - existing) then

  :update() — mailarchivemodel.php:321-374;

  partition "Load Existing Data" #LightBlue {
    :SELECT * FROM mail_archive
    WHERE ma_id = input.ma_id;
  }

  partition "Update Archive" #LightBlue {
    :UPDATE mail_archive SET
    semua field (sama seperti create)
    WHERE ma_id = input.ma_id;
  }

  partition "Handle Mail Reference Change" #LightBlue {
    if (old.ma_ref_id != new.ma_ref_id?) then (yes)

      if (old.ma_ref_id != 0?) then (yes)
        :Release old mail reference:
        UPDATE mail SET m_ma_id=NULL
        WHERE m_id = old.ma_ref_id;
      else (no)
      endif

      if (new.ma_ref_id != 0?) then (yes)
        :Book new mail reference:
        UPDATE mail SET m_ma_id=archive_id
        WHERE m_id = new.ma_ref_id;
      else (no)
      endif

    else (no, reference tidak berubah)
    endif
  }

  :Return success;

elseif (archive/finalize) then

  :archive() — mailarchivemodel.php:376-383;

  partition "Finalize" #LightBlue {
    :UPDATE mail_archive SET
    - ma_status = 2 (FINALIZED)
    - ma_archive_date = now()
    - ma_archive_by_name = user
    WHERE ma_id = id;
  }

  :Return success;

else (delete)

  :delete() — mailarchivemodel.php:385-399;

  partition "Soft Delete" #LightBlue {
    :SELECT * FROM mail_archive
    WHERE ma_id = id;

    :UPDATE mail_archive SET
    ma_status = 3 (DELETED)
    WHERE ma_id = id;

    if (ma_ref_id != 0?) then (yes)
      :Release mail reference:
      UPDATE mail SET m_ma_id=NULL
      WHERE m_id = ma_ref_id;
      note right
        Membebaskan surat agar
        bisa di-arsip ulang
      end note
    else (no)
    endif
  }

  :Return success;

endif

stop

@enduml
```

### Migration Notes
- Pisahkan menjadi method terpisah di `MailArchiveService`: create(), update(), finalize(), softDelete()
- Reference management → `@Transactional` untuk atomicity
- Validasi organisasi → call Kepegawaian API

---

## generate_code() — Auto Numbering Arsip

**Source:** `mailarchivemodel.php:435-551`
**Diagram type:** Activity
**Complexity:** High

### What
Generate nomor arsip per CLIENT_CODE dengan office-based branching. BMS dan SMD memiliki logic PUSAT vs cabang (prefix_sequence berbeda, m_cat ditambah office_code untuk cabang). BPN dan default tanpa office branching.

### Why
Nomor arsip harus unik per kantor dan per tahun. Cabang punya sequence terpisah agar numbering independen.

### Diagram

```plantuml
@startuml
!theme plain
title generate_code() — Auto Numbering Arsip Surat\nmailarchivemodel.php:435-551

start

if (CLIENT_CODE?) then (BMS)

  partition "BMS — Banyumas" #LightBlue {
    :ref_code = "ma_number_format_bms";
    :Load template dari sys_reference;
    :Load mail_code dari organization;
    :Replace #org_code# → mail_code;

    :Type mapping:
    1→"I/" (Internal)
    2→"M/" (Memo)
    3→"" (Eksternal, tanpa prefix);

    if (office_code == "PUSAT"\natau "ALL"?) then (PUSAT)
      :m_cat = ma_mcat_code;
      :prefix_sequence = type + YYYY;
    else (CABANG)
      :m_cat = ma_mcat_code + "-" + office_code;
      :prefix_sequence = office_code + "/" + type + YYYY;
      note right
        Cabang punya prefix terpisah
        agar sequence numbering
        independen per cabang
      end note
    endif

    :Replace #type#, #MR#, #YYYY#, #m_cat#;
    :sequence = get_trans_sequence(
    TASK_MAIL_ARCHIVE, prefix_sequence, **4**);
    note right
      Padding 4 digit (0001, 0002...)
      berbeda dengan mail yang 3 digit
    end note
    :Replace #seq# → sequence;
  }

elseif (SMD) then

  partition "SMD — Samarinda" #LightBlue {
    :ref_code = "ma_number_format_smd";
    note right
      Logic identik dengan BMS
      hanya beda ref_code template
    end note
    :Load template + mail_code;
    :Type mapping: 1→"I/", 2→"M/", 3→"";

    if (office_code == "PUSAT"/"ALL"?) then (PUSAT)
      :m_cat = ma_mcat_code;
      :prefix_sequence = type + YYYY;
    else (CABANG)
      :m_cat = ma_mcat_code + "-" + office_code;
      :prefix_sequence = office_code + "/" + type + YYYY;
    endif

    :Replace placeholders;
    :sequence = get_trans_sequence(
    TASK_MAIL_ARCHIVE, prefix_sequence, **4**);
    :Replace #seq#;
  }

elseif (BPN) then

  partition "BPN — Balikpapan" #LightBlue {
    :ref_code = "ma_number_format_blp";
    :Load template + mail_code;
    :Replace #org_code# → mail_code;

    :Type mapping (tanpa slash):
    1→"I" (Internal)
    2→"M" (Memo)
    3→"K" (Keluar);
    note right
      BPN: tanpa office branching.
      Tidak ada PUSAT vs cabang.
    end note

    :Replace #type#, #MR#, #YYYY#, #m_cat#;
    :prefix_sequence = type + "-"
    + ma_mcat_code + YYYY;
    :sequence = get_trans_sequence(
    TASK_MAIL_ARCHIVE, prefix_sequence, **3**);
    :Replace #seq#;
  }

else (default)

  partition "Default" #LightBlue {
    :ref_code = "ma_number_format_blp";
    :Type mapping: 1→"I", 2→"M", 3→"E";
    note right
      Default berbeda dari BPN:
      type 3 = "E" (bukan "K")
    end note
    :Logic sama seperti BPN
    (tanpa office branching);
    :sequence padding = **3** digit;
  }

endif

:Return kode arsip final;

stop

@enduml
```

### Migration Notes
- Strategy Pattern: `ArchiveCodeGenerator` per CLIENT_CODE
- Office branching → parameter di strategy, bukan if/else
- Padding 4 digit (BMS/SMD) vs 3 digit (BPN/default) → configurable

---

## Archive Access Control — Permission & Notification

**Source:** `mailarchivemodel.php:401-590`
**Diagram type:** Sequence
**Complexity:** Medium

### What
Position-based access control: set_access() delete-and-reinsert permissions per jabatan (3 level: access, download, history). Notification workflow: set_notif() queue → scheduled job picks up → get_allowed_user() resolves employees by position → cek_user_notif() prevents duplicate → create_archive_mail_notif() sends internal mail → set_user_notif() logs. Search enforces ACL via JOIN.

### Why
Arsip bersifat rahasia — akses berdasarkan jabatan (position), bukan individual. Notifikasi otomatis memastikan user yang diberi akses tahu ada arsip baru.

### Diagram

```plantuml
@startuml
!theme plain
title Archive Access Control — Permission & Notification Workflow\nmailarchivemodel.php:401-590

participant "Controller\n(mailarchive.php)" as Ctrl
participant "MailArchiveModel" as Model
participant "MailModel" as MailModel
database "Database" as DB
participant "Kepegawaian API\n(target migrasi)" as API

== Part 1: Set Access (set_access) ==
' mailarchivemodel.php:401-415

Ctrl -> Model : set_access(access[], archive_id)

Model -> DB : DELETE FROM mail_archive_access\nWHERE mail_archive_id = id
note right
  Delete-and-reinsert pattern.
  Semua permission di-reset lalu
  di-insert ulang dari input.
end note

loop untuk setiap access entry
  alt pos_id != 0
    Model -> DB : INSERT mail_archive_access\n(mail_archive_id, pos_id,\naccess=0/1, download=0/1, history=0/1)
    note right
      3 permission levels:
      - **access**: bisa lihat arsip
      - **download**: bisa download lampiran
      - **history**: bisa lihat riwayat
    end note
  else pos_id == 0
    Model -> Model : skip (invalid entry)
  end
end

== Part 2: Queue Notification (set_notif) ==
' mailarchivemodel.php:417-422

Ctrl -> Model : set_notif(archive_id)
Model -> DB : INSERT mail_archive_notif\n(mail_archive_id, notif_flag=0,\ninsert_date=now())
note right
  Flag 0 = belum diproses.
  Akan di-pickup oleh scheduled job.
end note

== Part 3: Notification Processing (Scheduled Job) ==
' mailarchivemodel.php:553-590

note over Model
  Berikut adalah flow yang dijalankan
  oleh scheduled job / cron:
end note

Model -> DB : get_unnotified()\nSELECT * FROM mail_archive_notif\nWHERE notif_flag=0
DB --> Model : unnotified list

loop untuk setiap unnotified archive
  Model -> DB : get_archive_by_id(archive_id)
  DB --> Model : archive object

  Model -> DB : get_allowed_user(archive_id)\nSELECT user_id, emp_id, emp_name...\nFROM mail_archive_access\nJOIN employee (by pos_id)\nJOIN emp_profile, sys_user, position\nWHERE user_status=1
  note right #Orange
    **Migrasi:** JOIN employee by pos_id.
    Di Spring Boot:
    GET /pegawai/{posId}/position
    untuk setiap pos_id di access list
  end note
  DB --> Model : allowed users[]

  loop untuk setiap allowed user
    Model -> DB : cek_user_notif(archive_id, user_id)\nSELECT COUNT FROM mail_archive_notif_log
    DB --> Model : count

    alt user belum pernah di-notif
      Model -> MailModel : create_archive_mail_notif(user, archive)
      note right
        Lihat diagram:
        **mail-notif-archive.puml**
      end note

      Model -> DB : set_user_notif(user_id, archive_id)\nINSERT mail_archive_notif_log\n(mail_archive_id, user_id, notif_date)
    else user sudah pernah di-notif
      Model -> Model : skip
    end
  end

  Model -> DB : mark_notified(archive_id)\nUPDATE mail_archive_notif\nSET notif_flag=1, processed_date=now()
end

== Part 4: Search with Access Control ==
' mailarchivemodel.php:123-261

note over Ctrl, DB
  Saat user melakukan search arsip,
  query JOIN mail_archive_access
  untuk filter berdasarkan pos_id user
  dan flag access=1
end note

Ctrl -> Model : search(params)
Model -> DB : SELECT ... FROM mail_archive ma\nJOIN mail_archive_access maa\nON maa.mail_archive_id=ma.ma_id\nWHERE maa.pos_id = user.pos_id\nAND maa.access = 1\nAND ma_status = 2
DB --> Model : filtered archives

@enduml
```

### Migration Notes
- set_access(): gunakan `@Transactional` + bulk insert (bukan delete-reinsert)
- Notification: Spring `@Scheduled` job atau event-driven
- ACL: `@PreAuthorize` custom atau Spring Security ACL module
- Employee lookup by position → Kepegawaian API: `GET /pegawai/{posId}/position`

---

