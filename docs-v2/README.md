# SmartOffice Migration Docs

> Dokumentasi migrasi **CodeIgniter 2.1.3 (Ext.Direct RPC)** → **Spring Boot REST API**
> Scope: Modul Persuratan (e-Office Mail)

---

## Navigasi Dokumen

| Dokumen | Deskripsi |
|---------|-----------|
| [00 — Feature Mapping](00-feature-mapping.md) | Pemetaan fungsi CI → Spring Boot REST endpoint |
| [01 — Non-Trivial Business Logic](01-non-trivial-business-logic.md) | Daftar logika bisnis kompleks yang butuh perhatian khusus |
| [02 — Migration Goals](02-migration-goals.md) | Tujuan, prinsip, dan scope migrasi |

---

## Business Logic Diagrams

Diagram alur bisnis per modul, dibuat untuk membantu memahami logika sebelum migrasi.

| Modul | File | Jumlah Diagram |
|-------|------|----------------|
| 📬 **Mail Core** | [business-logic/planuml/index-mail-core.md](business-logic/planuml/index-mail-core.md) | 9 |
| 🗄️ **Mail Archive** | [business-logic/planuml/index-mail-archive.md](business-logic/planuml/index-mail-archive.md) | 3 |
| 🔔 **Mail Support & Notifications** | [business-logic/planuml/index-mail-support.md](business-logic/planuml/index-mail-support.md) | 4 |

---

## Mail Core — Ringkasan Diagram

| # | Fungsi | Source | Tipe | Kompleksitas |
|---|--------|--------|------|--------------|
| 1 | `send()` — Pengiriman Surat (10 side-effects) | `mailmodel.php:812` | Activity | Very High |
| 2 | `generate_code()` — Auto Numbering Surat | `mailmodel.php:1564` | Activity | High |
| 3 | `readFolder()` — Baca Surat per Folder | `mailmodel.php:83` | Activity | High |
| 4 | `make_nlevel_threaded()` — Thread Tree Builder | `mail.php:219` | Activity | Medium |
| 5 | `find()` — Pencarian Surat | `mailmodel.php:1219` | Activity | Medium |
| 6 | `directArsip()` — Arsip Langsung (3-gate validation) | `mail.php:364` | Activity | Medium |
| 7 | `signMe()` + `checkSign()` — Verifikasi Cetak | `mail.php:395` | Activity | Low |
| 8 | `trackMail()` — Pelacakan Sirkulasi | `mailmodel.php:1174` | Activity | Medium |
| 9 | Mail Folder Lifecycle (move/restore/empty_trash) | `mailmodel.php:1018` | State | Medium |

---

## Mail Archive — Ringkasan Diagram

| # | Fungsi | Source | Tipe | Kompleksitas |
|---|--------|--------|------|--------------|
| 1 | `create()` + `update()` + `archive()` + `delete()` | `mailarchivemodel.php:263` | Activity | High |
| 2 | `generate_code()` — Auto Numbering Arsip | `mailarchivemodel.php:435` | Activity | High |
| 3 | Access Control + Notification Workflow | `mailarchivemodel.php:401` | Sequence | Medium |

---

## Mail Support — Ringkasan Diagram

| # | Fungsi | Source | Tipe | Kompleksitas |
|---|--------|--------|------|--------------|
| 1 | `create_publication_mail_notif()` — Broadcast ke semua karyawan | `mailmodel.php:337` | Sequence | High |
| 2 | `create_upd_emp_tanggungan_mail_notif()` — Notif perubahan tanggungan | `mailmodel.php:444` | Sequence | Medium |
| 3 | `create_approval_cuti_notif()` — Notif approval cuti | `mailmodel.php:572` | Sequence | Medium |
| 4 | `create_archive_mail_notif()` — Notif hak akses arsip | `mailmodel.php:1617` | Sequence | Medium |

---

## PlantUML Source Files

File `.puml` tersedia di [`business-logic/planuml/puml/`](business-logic/planuml/puml/) untuk digunakan dengan PlantUML extension atau tools lokal.

> **Catatan:** Diagram dalam file index sudah dirender sebagai gambar menggunakan [PlantUML Server](https://www.plantuml.com/). Tidak perlu install extension untuk melihat diagram di GitHub.
