# 4. Entity-Relationship Mapping

## 4.1 Database → REST Resource Mapping

```
┌─────────────────────┐          ┌──────────────────────┐
│      mail            │          │   mail_recipient     │
│  (REST: /mails)      │ 1────M  │ (/mails/{id}/        │
│                      │          │    recipients)        │
│  m_id (PK)           │          │  mail_id (FK)        │
│  m_no                │          │  user_id             │
│  m_date              │          │  emp_id → [HR Svc]   │
│  m_root_id ──┐       │          │  emp_name (cached)   │
│  m_parent_id │thread  │          │  pos_id → [HR Svc]   │
│  m_type (FK) │       │          │  pos_name (cached)   │
│  m_category (FK)     │          │  circulation         │
│  m_subject           │          │  email (notif flag)  │
│  m_content           │          │  sms (notif flag)    │
│  m_note              │          └──────────────────────┘
│  m_max_response_date │
│  m_status            │          ┌──────────────────────┐
│  m_created_by → user │ 1────M  │  sys_user_task       │
│  m_created_by_name   │          │  (internal, no REST) │
│  m_attachment_qty    │          │                      │
│  m_to_str (cached)   │          │  user_id             │
│  m_no_surat_masuk    │          │  tm_id = mail_id     │
│  m_asal_surat_masuk  │          │  folder_id           │
│  m_tgl_surat_masuk   │          │  read_status         │
│  m_tujuan_surat_keluar│         │  restore_folder_id   │
│  m_penerima_surat_keluar│       └──────────────────────┘
└─────────────────────┘
         │
    M────1                        ┌──────────────────────┐
         │                        │   mail_archive       │
┌────────┴──────────┐             │  (REST: /archives)   │
│   mail_type       │             │                      │
│ (/mail-types)     │             │  ma_id (PK)          │
│                   │             │  ma_no               │
│  mail_type_id(PK) │    1────M  │  ma_ref_id → mail.m_id│
│  mail_type        │◄───────────│  ma_mcat_type (FK)   │
│  mail_type_status │             │  ma_mcat_id (FK)     │
└───────────────────┘             │  ma_org_id → [HR]    │
         │                        │  ma_subject          │
    1────M                        │  ma_content          │
         │                        │  ma_secret_type      │
┌────────┴──────────┐             │  ma_loc_* (fisik)    │
│  mail_category    │             │  ma_keyword          │
│ (/mail-categories)│             │  ma_status           │
│                   │             │  office_code         │
│  mcat_id (PK)     │             └──────────────────────┘
│  mail_type_id(FK) │                      │
│  mcat_code        │                 1────M
│  mcat_name        │                      │
│  mcat_status      │             ┌──────────────────────┐
└───────────────────┘             │ mail_archive_access  │
                                  │ (/archives/{id}/     │
┌───────────────────┐             │         access)      │
│  mail_folder      │             │                      │
│ (REST: /folders)  │             │  mail_archive_id(FK) │
│                   │             │  pos_id → [HR Svc]   │
│  folder_id (PK)   │             │  access (0/1)        │
│  parent_folder_id │             │  download (0/1)      │
│  owner_id (user)  │             │  history (0/1)       │
│  folder_name      │             └──────────────────────┘
│  folder_status    │
└───────────────────┘             ┌──────────────────────┐
                                  │  attachments         │
                                  │ (/mails/{id}/        │
                                  │   attachments)       │
                                  │ (/archives/{id}/     │
                                  │   attachments)       │
                                  │                      │
                                  │  ref_type (1=mail,   │
                                  │           2=archive) │
                                  │  ref_id              │
                                  │  file_ext            │
                                  │  file_size           │
                                  │  original_filename   │
                                  │  system_filename     │
                                  └──────────────────────┘
```

## 4.2 Status & Enum Values

**Mail Status (`m_status`):**

| Value | Const | API Enum |
|---|---|---|
| 0 | MAIL_STATUS_DRAFT | `DRAFT` |
| 1 | MAIL_STATUS_SENT | `SENT` |
| 2 | MAIL_STATUS_DELETED | `DELETED` |

**Folder IDs (System Folders):**

| ID | Const | API Enum |
|---|---|---|
| 2 | FOLDER_INBOX | `INBOX` |
| 3 | FOLDER_DRAFT | `DRAFT` |
| 4 | FOLDER_READ | `READ_ITEMS` |
| 5 | FOLDER_SENT | `SENT` |
| 6 | FOLDER_DELETED | `TRASH` |
| >10 | FOLDER_PERSONAL | Custom folders |

**Circulation Types:**

| Value | Keterangan | API Enum |
|---|---|---|
| 1 | Kepada (To) | `TO` |
| 2 | Tembusan (CC) | `CC` |
| 3 | Disposisi | `DISPOSITION` |
| 4 | Informasi/Broadcast | `INFO` |
| 5 | Reply | `REPLY` |

**Mail Types:**

| ID | Name | API Enum |
|---|---|---|
| 1 | Internal | `INTERNAL` |
| 2 | Masuk (Incoming) | `INCOMING` |
| 3 | Keluar (Outgoing) | `OUTGOING` |

**Archive Status:**

| Value | Keterangan | API Enum |
|---|---|---|
| 1 | Draft | `DRAFT` |
| 2 | Archived | `ARCHIVED` |
| 3 | Deleted | `DELETED` |

<!-- nav -->
---
**Navigation:** [< Prev](03-common-conventions.md) | [Next >](06-hr-service-integration.md)
<!-- /nav -->
