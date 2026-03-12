# SmartOffice — Migration Documentation

Dokumentasi migrasi sistem **SmartOffice** dari **CodeIgniter 2.1.3 (Ext.Direct RPC)** ke **Spring Boot REST API**.

## Struktur Dokumen

| Folder | Deskripsi |
|--------|-----------|
| [`docs-v2/`](docs-v2/) | Dokumentasi versi terbaru (aktif) |
| [`out/`](out/) | Output / artefak hasil proses |

## Mulai dari Sini

👉 **[Buka docs-v2/README.md](docs-v2/README.md)** untuk navigasi lengkap.

---

### Ringkasan Scope

- **Source:** CodeIgniter 2.1.3, Ext.Direct RPC, ExtJS frontend
- **Target:** Spring Boot REST API, OpenAPI spec, JWT auth
- **Modul:** Persuratan (e-Office Mail) — Mail Core, Mail Archive, Mail Category, Mail Type

### Dokumen Utama

| # | Dokumen | Deskripsi |
|---|---------|-----------|
| — | [Feature Mapping](docs-v2/00-feature-mapping.md) | CI function → REST endpoint mapping |
| — | [Non-Trivial Business Logic](docs-v2/01-non-trivial-business-logic.md) | Logika kompleks yang perlu perhatian |
| — | [Migration Goals](docs-v2/02-migration-goals.md) | Tujuan dan prinsip migrasi |
| — | [Mail Core Diagrams](docs-v2/business-logic/planuml/index-mail-core.md) | 9 diagram alur bisnis mail |
| — | [Mail Archive Diagrams](docs-v2/business-logic/planuml/index-mail-archive.md) | 3 diagram alur arsip |
| — | [Mail Support Diagrams](docs-v2/business-logic/planuml/index-mail-support.md) | 4 diagram notifikasi sistem |
