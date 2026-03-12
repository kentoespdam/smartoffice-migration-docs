# Tujuan Migrasi: CI 2.1.3 RPC → Spring Boot REST API

> **Modul:** Persuratan (e-Office Mail) &nbsp;|&nbsp; **Stack Lama:** CodeIgniter 2.1.3 + Ext.Direct RPC &nbsp;|&nbsp; **Stack Baru:** Spring Boot REST + JWT

---

## Quick Reference

| | Sistem Lama | Sistem Baru |
|---|---|---|
| **Protokol** | Ext.Direct RPC (JSON-RPC) | REST (HTTP verbs + resource URLs) |
| **Auth** | Session PHP `$_SESSION` | JWT / OAuth2 stateless |
| **Business Logic** | Fat Model PHP (~2000 baris) | `@Service` + `@Transactional` |
| **Notifikasi** | Synchronous per-request | `@Async` / `@EventListener` |
| **Data Pegawai** | SQL JOIN lokal | HTTP → API Kepegawaian (cached) |
| **Kontrak API** | Tidak formal | OpenAPI / Swagger |
| **Testing** | Tight coupling, sulit | Unit + Integration test (Spring Test) |
| **Scalability** | Single PHP monolith | Modular / microservice-ready |

**API Kepegawaian:** `http://192.168.1.214:8080` &nbsp;|&nbsp; Docs: `/v3/api-docs` &nbsp;|&nbsp; Auth: JWT Bearer wajib

---

## 1. Scope Migrasi

**Modul:** Persuratan (e-Office Mail)

| Sub-modul | CI File | CI Model | Spring Boot Target |
|---|---|---|---|
| Surat | `mail.php` | `mailmodel.php` | `MailController` + `MailService` |
| Arsip Surat | `mailarchive.php` | `mailarchivemodel.php` | `MailArchiveController` + `MailArchiveService` |
| Kategori Surat | `mailcategory.php` | `mailcategorymodel.php` | `MailCategoryController` |
| Jenis Surat | `mailtype.php` | `mailtypemodel.php` | `MailTypeController` |
| Laporan Statistik | `mailstatisticreport.php` | — | `MailStatisticController` |
| Attachment | `attachment.php` | `attachmentmodel.php` | `AttachmentController` |

---

## 2. Prinsip Migrasi

1. **Dokumentasi dulu** — jangan tulis ulang logika bisnis tanpa memahaminya
2. **Backward compatibility** — field denormalized (`emp_name`, `pos_name`) tetap diisi
3. **Decouple notifikasi** — pisahkan dari transaksi utama via `@Async` / Spring Events
4. **Multi-tenant** — gunakan Strategy Pattern, jangan hardcode `if (BMS)` / `else if (SMD)`
5. **Employee data** — selalu dari API terpusat + `@Cacheable`, bukan DB lokal

---

## 3. Integrasi API Kepegawaian

> ⚠️ **Breaking change:** Data kepegawaian tidak lagi di-JOIN dari DB lokal. Semua diambil dari API eksternal dengan autentikasi JWT.

### 3.1 Konfigurasi Koneksi

```
Base URL : http://192.168.1.214:8080
API Docs : http://192.168.1.214:8080/v3/api-docs  (OpenAPI 3.0.1)
Auth     : Bearer JWT — wajib di semua endpoint
Kontak   : Kent-Os (kentoes.pdam@gmail.com)
```

**Ambil token via:**
- `GET /auth/session` — session token
- `GET /auth/csrf-token` — CSRF token

<details>
<summary><strong>Implementasi RestTemplate dengan JWT Interceptor</strong></summary>

```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
    return builder
        .interceptors((request, body, execution) -> {
            request.getHeaders().set("Authorization", "Bearer " + jwtToken);
            return execution.execute(request, body);
        })
        .build();
}
```

</details>

### 3.2 Endpoint yang Tersedia

<details>
<summary><strong>Pegawai <code>/pegawai</code></strong></summary>

| Endpoint | Method | Fungsi |
|---|---|---|
| `/pegawai` | `GET` | Daftar pegawai (dengan paginasi) |
| `/pegawai` | `POST` | Tambah pegawai baru |
| `/pegawai/list` | `GET` | Daftar pegawai (tanpa paginasi) |
| `/pegawai/batch` | `POST` | Import batch pegawai |
| `/pegawai/{id}` | `GET` | Detail pegawai by ID |
| `/pegawai/{id}/profil` | `GET` | Profil lengkap pegawai |
| `/pegawai/{id}/ringkasan` | `GET` | Ringkasan pegawai |
| `/pegawai/{id}/gaji` | `GET` | Data gaji pegawai |
| `/pegawai/{nipam}/nipam` | `GET` | Cari pegawai by NIPAM |

> ⚠️ Tidak ada filter query param (`positionId`, `orgId`, `status`). Filtering dilakukan **di sisi aplikasi**.

</details>

<details>
<summary><strong>Jabatan <code>/master/jabatan</code></strong></summary>

| Endpoint | Method | Fungsi |
|---|---|---|
| `/master/jabatan` | `GET` | Daftar jabatan |
| `/master/jabatan/list` | `GET` | Daftar jabatan (tanpa paginasi) |
| `/master/jabatan/batch` | `POST` | Import batch jabatan |
| `/master/jabatan/{id}` | `GET` | Detail jabatan by ID |
| `/master/jabatan/organisasi/{id}` | `GET` | Jabatan dalam unit kerja tertentu |

</details>

<details>
<summary><strong>Unit Kerja / Organisasi</strong></summary>

Tidak ada endpoint organisasi terpisah. Data organisasi tersedia di:
- Field `organisasi` / `unitKerja` dalam response object `pegawai`
- Endpoint `/master/jabatan/organisasi/{id}` untuk jabatan per unit kerja

</details>

### 3.3 Mapping SQL Join → API Call

| Konteks | Query Lama | API Baru |
|---|---|---|
| Autocomplete penerima | `JOIN employee, emp_profile, position` | `GET /pegawai` → filter di aplikasi |
| `addRecipient` | Resolve `emp_id` dari form | `GET /pegawai/{nipam}/nipam` |
| Notif kirim surat | `JOIN employee, emp_profile` (email) | `GET /pegawai/{id}/profil` |
| Notif arsip (`get_allowed_user`) | `JOIN employee WHERE emp_pos_id = pos_id` | `GET /master/jabatan/organisasi/{orgId}` |
| Notif publikasi (semua karyawan) | `SELECT * FROM sys_user JOIN employee` | `GET /pegawai/list` |
| Notif cuti / tanggungan | `JOIN employee WHERE emp_id = {id}` | `GET /pegawai/{id}` |
| Tracking surat (sender) | `JOIN employee, position` | Field `position` dari objek pegawai (cached) |
| Read folder (sender name) | `JOIN sys_user` | Kolom denormalized `m_created_by_name` di tabel `mail` |

> ⚠️ **Denormalized fields wajib dipertahankan:** `emp_name`, `pos_name` di `mail_recipient` dan `m_created_by_name` di `mail` — jangan diubah ke JOIN agar data historis tetap konsisten.

### 3.4 Arsitektur Integrasi

```
MailService
├── addRecipient()  →  KepegawaianClient.getByNipam()
└── send()          →  KepegawaianClient.getProfil()

KepegawaianClient  (@Cacheable, TTL 1–6 jam)
├── GET /pegawai/{nipam}/nipam
├── GET /pegawai/{id}/profil
└── GET /master/jabatan/organisasi/{id}
```

---

## 4. Penyesuaian Desain API (Deviasi dari Rencana Awal)

> ⚠️ API aktual **berbeda** dari yang direncanakan semula.

<details>
<summary><strong>Lihat detail perbedaan</strong></summary>

| Rencana Awal | Kenyataan API Aktual |
|---|---|
| `/pegawai?nama=x&status=ACTIVE` | `/pegawai` — tidak ada filter param |
| `/pegawai/{posId}/position` | Tidak ada — gunakan `/master/jabatan/organisasi/{id}` |
| Endpoint `/organization` terpisah | Tidak ada — organisasi hanya di field pegawai |

**Implikasi:**
- **Filtering & sorting** dilakukan in-memory di Spring Boot
- **Data organisasi** diambil dari field denormalized di response pegawai
- **Caching wajib** — `@Cacheable` di `KepegawaianClient` dengan TTL 1–6 jam
- **Background sync** direkomendasikan untuk pre-load data pegawai

</details>

### Action Items

- [ ] Desain caching strategy (Redis vs Caffeine)
- [ ] Implementasi `KepegawaianClient` dengan Feign + `@Cacheable`
- [ ] Background job untuk periodic sync pegawai
- [ ] Update `MailService.addRecipient()` untuk pakai cached data
- [ ] Test performa autocomplete dengan dataset besar

---

## Referensi

| Dokumen | Deskripsi |
|---|---|
| [00-feature-mapping.md](00-feature-mapping.md) | Peta fungsi RPC → REST endpoint |
| [01-non-trivial-business-logic.md](01-non-trivial-business-logic.md) | Analisis logika bisnis kompleks |
| [business-logic/](business-logic/) | Flowchart logika per fungsi per modul |
| [API Docs Live](http://192.168.1.214:8080/v3/api-docs) | OpenAPI 3.0.1 — Kepegawaian API |
