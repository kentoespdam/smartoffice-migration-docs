# Tujuan Migrasi: CI 2.1.3 RPC → Spring Boot REST API

## Latar Belakang

Sistem SmartOffice saat ini menggunakan **CodeIgniter 2.1.3** dengan pola komunikasi **Ext.Direct RPC** — frontend ExtJS memanggil method PHP secara langsung via JSON-RPC. Pola ini tightly-coupled, tidak stateless, dan sulit di-scale atau di-test secara terpisah.

---

## Tujuan Migrasi

| Aspek | Sistem Lama (CI + RPC) | Sistem Baru (Spring Boot REST) |
|---|---|---|
| **Protokol** | Ext.Direct RPC (method-based JSON) | REST API (HTTP verbs + resource-based URL) |
| **Kontrak API** | Tidak ada kontrak formal, tergantung frontend | OpenAPI / Swagger spec yang eksplisit |
| **Autentikasi** | Session PHP (`$_SESSION`) | JWT / OAuth2 stateless |
| **Business Logic** | Fat Model PHP (mailmodel.php ~2000 baris) | Service layer terpisah dengan `@Transactional` |
| **Notifikasi** | Synchronous dalam satu request | `@Async` / Event-driven (`@EventListener`) |
| **Testability** | Sulit unit test (tight coupling CI) | Unit + Integration test mudah (Spring Test) |
| **Data Kepegawaian** | Join langsung ke tabel `employee` di DB lokal | HTTP Client ke API Kepegawaian eksternal |
| **Scalability** | Single monolith PHP | Modular Spring Boot (bisa microservice) |

---

## Scope Migrasi Saat Ini

**Modul yang dimigrasikan**: Persuratan (e-Office Mail)

| Sub-modul | CI File (direct/) | CI Model | Spring Boot Target |
|---|---|---|---|
| Surat (Mail) | mail.php | mailmodel.php | `MailController` + `MailService` |
| Arsip Surat | mailarchive.php | mailarchivemodel.php | `MailArchiveController` + `MailArchiveService` |
| Kategori Surat | mailcategory.php | mailcategorymodel.php | `MailCategoryController` |
| Jenis Surat | mailtype.php | mailtypemodel.php | `MailTypeController` |
| Laporan Statistik | mailstatisticreport.php | — | `MailStatisticController` |
| Attachment | attachment.php | attachmentmodel.php | `AttachmentController` |

---

## Sumber Data Kepegawaian

### ⚠️ Perubahan Penting: Employee Data dari API Eksternal

Pada sistem lama, data kepegawaian diambil via **join SQL langsung** ke tabel `employee`, `emp_profile`, `position`, dan `organization` dalam database yang sama.

Pada sistem baru, **data kepegawaian TIDAK tersimpan di database lokal SmartOffice**. Semua data karyawan, jabatan, dan organisasi diambil dari **API Kepegawaian Terpusat**:

```
Base URL: http://192.168.1.183:8088
Docs:     http://192.168.1.183:8088/api-docs (OpenAPI 3.0.1)
```

### Endpoint API Kepegawaian yang Tersedia

#### Pegawai (`/pegawai`)
| Endpoint | Method | Fungsi |
|---|---|---|
| `/pegawai` | GET | Daftar pegawai (filter: nipam, nama, positionId, orgId, status) |
| `/pegawai/page` | GET | Daftar pegawai dengan paginasi |
| `/pegawai/{nipam}/nipam` | GET | Cari pegawai by NIPAM |
| `/pegawai/{nipams}/in` | GET | Cari banyak pegawai by list NIPAM |
| `/pegawai/{posId}/position` | GET | Semua pegawai di satu jabatan |
| `/pegawai/{posParent}/staff` | GET | Staff di bawah satu jabatan parent |
| `/pegawai/{orgCode}/organization` | GET | Semua pegawai di satu unit kerja |

**Filter parameter `PegawaiRequest`:**
```
nipam, nama, positionId, positionParent, organizationId,
flagStatusId, workStatusId, status: [NONE | ACTIVE | INACTIVE | DELETED]
```

#### Jabatan (`/positions`)
| Endpoint | Method | Fungsi |
|---|---|---|
| `/positions` | GET | Daftar jabatan (filter: name, code, level, parent, status, orgId) |
| `/positions/{id}` | GET | Detail jabatan by ID |
| `/positions/list` | GET | Daftar jabatan (tanpa paginasi) |
| `/positions/in/{ids}` | GET | Banyak jabatan by list ID |

#### Organisasi / Unit Kerja (`/organization`)
| Endpoint | Method | Fungsi |
|---|---|---|
| `/organization` | GET | Daftar organisasi (filter: name, code, level, parent, group, category) |
| `/organization/{id}` | GET | Detail organisasi by ID |
| `/organization/list` | GET | Daftar organisasi (tanpa paginasi) |
| `/organization/in/{ids}` | GET | Banyak organisasi by list ID |

---

### Mapping: Penggunaan Data Kepegawaian di Modul Persuratan

Berikut adalah titik-titik di mana sistem lama JOIN tabel kepegawaian, dan mapping ke API baru:

| Konteks Penggunaan | Query Lama (SQL Join) | API Baru |
|---|---|---|
| Daftar penerima surat (autocomplete) | `JOIN employee e, emp_profile ep, position p` | `GET /pegawai?nama={keyword}&status=ACTIVE` |
| Tambah penerima (`addRecipient`) | Resolve emp_id dari form | `GET /pegawai/{nipam}/nipam` untuk validasi |
| Notifikasi kirim surat (send) | `JOIN employee e, emp_profile ep` untuk email | Cache lokal dari `GET /pegawai/{nipam}/nipam` |
| Notifikasi arsip (`get_allowed_user`) | `JOIN employee e WHERE emp_pos_id = pos_id` | `GET /pegawai/{posId}/position` |
| Publikasi mail notif (semua karyawan) | `SELECT * FROM sys_user JOIN employee` | `GET /pegawai?status=ACTIVE` (semua aktif) |
| Cuti notif, tanggungan notif | `JOIN employee WHERE emp_id = {id}` | `GET /pegawai/{nipam}/nipam` |
| Tracking surat (pos/org sender) | `JOIN employee ex, position p` | Cache dari sesi login user |
| Read folder (sender info) | `JOIN sys_user u` (nama disimpan di mail.m_created_by_name) | Tetap dari kolom denormalized |

### Strategi Integrasi di Spring Boot

```
┌─────────────────────────────────────────────────────────┐
│                   Spring Boot Service                    │
│                                                          │
│  MailService                                             │
│    └── saat addRecipient() → panggil KepegawaianClient  │
│    └── saat send()        → resolve email dari API       │
│                                                          │
│  KepegawaianClient (Feign / RestTemplate / WebClient)   │
│    └── GET http://192.168.1.183:8088/pegawai/{nipam}/nipam │
│    └── GET http://192.168.1.183:8088/positions/{id}      │
│    └── Hasil di-cache (@Cacheable) untuk performance     │
└─────────────────────────────────────────────────────────┘
```

> ⚠️ **Catatan penting**: Field `emp_name`, `pos_name` di tabel `mail_recipient` dan `mail.m_created_by_name` tetap **disimpan secara denormalized** di DB SmartOffice (tidak join saat read) — ini adalah pola existing yang HARUS dipertahankan agar historical data tetap konsisten walaupun data jabatan berubah.

---

## Prinsip Migrasi

1. **Tidak menulis ulang logika bisnis secara sembarangan** — dokumentasikan dulu, baru implementasi
2. **Pertahankan backward compatibility** untuk data lama (field denormalized tetap diisi)
3. **Decoupling notifikasi** dari transaksi utama via `@Async` / Spring Events
4. **Multi-tenant (CLIENT_CODE)** via Strategy Pattern — jangan hardcode `if (BMS)`, `else if (SMD)`
5. **Employee data** dari API terpusat, bukan dari DB lokal — pakai HTTP Client dengan caching

---

## Referensi Dokumen Terkait

- [00-feature-mapping.md](00-feature-mapping.md) — Peta fungsi RPC ke REST endpoint
- [01-non-trivial-business-logic.md](01-non-trivial-business-logic.md) — Analisis logika bisnis kompleks
- [business-logic/](business-logic/) — Flowchart logika per fungsi per modul
