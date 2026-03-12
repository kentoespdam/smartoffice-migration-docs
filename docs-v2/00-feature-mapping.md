# Feature Mapping: CI 2.1.3 (Ext Direct RPC) -> Spring Boot REST API

## Source Files Overview

| CI Layer | Files |
| --- | --- |
| RPC Direct | `mail.php`, `mailarchive.php`, `mailcategory.php`, `mailtype.php`, `mailstatisticreport.php` |
| Models | `mailmodel.php`, `mailarchivemodel.php`, `mailcategorymodel.php`, `mailtypemodel.php`, `mailstatisticreportmodel.php` |
| Frontend (ExtJS) | Controllers: `Mail.js`, `MailArchive.js`, `MailCategory.js`, `MailStandard.js`, etc. |

## 1. Mail (Surat) - Core Module

| # | CI Function (`direct/mail.php`) | Spring Boot REST Endpoint | Description |
| --- | --- | --- | --- |
| 1 | `getFolder()` | `GET /api/mail/folders` | List mail folders |
| 2 | `saveFolder($data)` | `POST /api/mail/folders` | Create/update folder |
| 3 | `delFolder($id)` | `DELETE /api/mail/folders/{id}` | Delete folder |
| 4 | `readFolder($input)` | `GET /api/mail/folders/{id}/mails` | Read mails in folder (inbox/sent/draft/trash) |
| 5 | `save($data)` | `POST /api/mails` or `PUT /api/mails/{id}` | Create/update mail (draft or send) |
| 6 | `delMail($id, $folderId)` | `DELETE /api/mails/{id}` | Delete/trash mail |
| 7 | `setRead($id, $folder)` | `PATCH /api/mails/{id}/read` | Mark mail as read |
| 8 | `flagRead($tid)` | `PATCH /api/mails/{tid}/flag` | Flag read status on thread |
| 9 | `addRecipient(...)` | `POST /api/mails/{id}/recipients` | Add single recipient |
| 10 | `addMultiRecipients(...)` | `POST /api/mails/{id}/recipients/bulk` | Add multiple recipients |
| 11 | `delRecipient($id)` | `DELETE /api/mails/{mailId}/recipients/{id}` | Remove recipient |
| 12 | `copyRecipient(...)` | `POST /api/mails/{id}/recipients/copy` | Copy recipients from another mail |
| 13 | `Recipient($input)` | `GET /api/mails/{id}/recipients` | List recipients |
| 14 | `trackMail($input)` | `GET /api/mails/{id}/tracking` | Track mail thread/disposisi tree |
| 15 | `find($input)` | `GET /api/mails/search` | Search mails |
| 16 | `move($data)` | `PATCH /api/mails/{id}/move` | Move mail to folder |
| 17 | `restore($id)` | `PATCH /api/mails/{id}/restore` | Restore from trash |
| 18 | `empty_trash()` | `DELETE /api/mail/trash` | Empty trash |
| 19 | `getcounter()` | `GET /api/mail/counters` | Unread counts per folder |
| 20 | `read($input)` | `GET /api/mails/{id}` | Read single mail detail |
| 21 | `getReadStatus($input)` | `GET /api/mails/{id}/read-status` | Who has read |
| 22 | `getPesan($input)` | `GET /api/mails/{id}/message` | Get mail body/content |
| 23 | `directArsip($id)` | `POST /api/mails/{id}/archive` | Direct archive mail |
| 24 | `signMe($id)` | `POST /api/mails/{id}/sign` | Digital signature |
| 25 | `checkSign($key)` | `GET /api/mails/verify-sign/{key}` | Verify signature |

### Model Mapping (`mailmodel.php`)

- `generate_code()` -> Service layer (auto-generate mail number)
- `send()` -> Service layer (send logic with notifications)
- `create_*_notif()` -> Notification service (publication, cuti, tanggungan)
- `export_xls()` -> `GET /api/mails/export?format=xlsx`

## 2. Mail Archive (Arsip Surat)

| # | CI Function (`direct/mailarchive.php`) | Spring Boot REST Endpoint |
| --- | --- | --- |
| 1 | `read($params)` | `GET /api/mail-archives` |
| 2 | `readAccess($params)` | `GET /api/mail-archives/{id}/access` |
| 3 | `search($params)` | `GET /api/mail-archives/search` |
| 4 | `save($data)` | `POST /api/mail-archives` |
| 5 | `archive($id)` | `PATCH /api/mail-archives/{id}/archive` |
| 6 | `del($id)` | `DELETE /api/mail-archives/{id}` |
| 7 | `report($params)` | `GET /api/mail-archives/report` |

## 3. Mail Category (Klasifikasi Surat)

| # | CI Function | Spring Boot REST Endpoint |
| --- | --- | --- |
| 1 | `read($input)` | `GET /api/mail-categories` |
| 2 | `save($data)` | `POST /api/mail-categories` |
| 3 | `del($id)` | `DELETE /api/mail-categories/{id}` |

## 4. Mail Type (Jenis Surat)

| # | CI Function | Spring Boot REST Endpoint |
| --- | --- | --- |
| 1 | `read($input)` | `GET /api/mail-types` |
| 2 | `read_tree()` | `GET /api/mail-types/tree` |
| 3 | `save($data)` | `POST /api/mail-types` |
| 4 | `del($id)` | `DELETE /api/mail-types/{id}` |

## 5. Mail Statistic Report

| # | CI Function | Spring Boot REST Endpoint |
| --- | --- | --- |
| 1 | `getStatistic($input)` | `GET /api/reports/mail-statistics` |

## Proposed Spring Boot Package Structure

```text
com.smartoffice.persuratan
├── controller/
│   ├── MailController.java          (25 endpoints)
│   ├── MailArchiveController.java   (7 endpoints)
│   ├── MailCategoryController.java  (3 endpoints)
│   ├── MailTypeController.java      (4 endpoints)
│   └── MailReportController.java    (2 endpoints)
├── service/
│   ├── MailService.java             (core business logic from mailmodel.php)
│   ├── MailArchiveService.java
│   ├── MailNotificationService.java (extracted from create_*_notif methods)
│   └── MailCodeGeneratorService.java
├── repository/
│   ├── MailRepository.java
│   ├── MailFolderRepository.java
│   ├── MailRecipientRepository.java
│   ├── MailArchiveRepository.java
│   ├── MailCategoryRepository.java
│   └── MailTypeRepository.java
├── model/entity/
│   ├── Mail.java, MailFolder.java, MailRecipient.java
│   ├── MailArchive.java, MailCategory.java, MailType.java
└── model/dto/
    ├── MailRequest.java, MailResponse.java, etc.
```

## Summary

| Module | CI Functions | REST Endpoints | Priority |
| --- | --- | --- | --- |
| Mail (Core) | ~25 | ~25 | High |
| Mail Archive | 7 | 7 | Medium |
| Mail Category | 3 | 3 | Low |
| Mail Type | 4 | 4 | Low |
| Mail Report | 1+ | 2 | Medium |
| Total | ~41 | ~41 | - |