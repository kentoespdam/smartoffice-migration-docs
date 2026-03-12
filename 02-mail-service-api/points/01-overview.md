# 1. Overview

Mail Service mengelola **persuratan internal** (nota dinas, memo, disposisi) dalam organisasi. Service ini merupakan hasil migrasi dari modul mail pada sistem legacy SmartOffice (PHP CI2).

## Functional Scope

| Capability | Deskripsi |
|---|---|
| **Mail Composition** | Buat draft, edit, kirim surat dengan recipients & attachments |
| **Folder Management** | Inbox, Sent, Draft, Read Items, Trash, Custom Folders |
| **Threading** | Surat bersifat threaded (reply/disposisi membentuk chain) |
| **Mail Archive** | Arsip surat dengan lokasi fisik, access control, dan keyword search |
| **Notification** | Email notification saat surat dikirim (async via message queue) |
| **Reporting** | Statistik surat per kategori, per organisasi, response time |
| **Number Generation** | Auto-generate nomor surat berdasarkan template configurable |

## Tech Stack Target

| Component | Technology |
|---|---|
| Runtime | Spring Boot 3.x |
| Database | MariaDB (existing, shared schema phase 1 → dedicated phase 2) |
| Auth | AppWrite v1.3.4 (JWT validation) |
| File Storage | MinIO/S3-compatible (migrasi dari filesystem) |
| Message Queue | RabbitMQ (notification, async processing) |
| API Doc | OpenAPI 3.0 (generated dari code) |

<!-- nav -->
---
**Navigation:** [< Prev](../README.md) | [Next >](02-authentication-security.md)
<!-- /nav -->
