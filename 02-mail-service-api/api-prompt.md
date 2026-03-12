# ROLE
Senior Software Architect & API Designer specializing in Spring Boot and Microservices Migration.

# CONTEXT
Migrasi sistem legacy "SmartOffice" dari **PHP CodeIgniter 2 (RPC Model)** ke arsitektur modern **Spring Boot REST API**. Sistem akan didekomposisi menjadi layanan terpisah (Decoupled Services).

# Current Infrastructure & Integration:
- Database: MariaDB (192.168.230.84:3307, db: smartoffice).
- Authentication: AppWrite v1.3.4 (Self-hosted) sebagai Identity Provider dan Session Management.
- External Integration: Harus berinteraksi dengan HR Service (Kepegawaian) via REST API (http://192.168.1.214:8080/v3/api-docs) untuk data personil/struktur organisasi.

# OBJECTIVE
Menganalisis skema database (smartoffice.sql) dan logic pada CI2 (Controller/Model) untuk menghasilkan **Dokumentasi Rancangan REST API (API Contract)** yang akan digunakan sebagai panduan utama (Single Source of Truth) oleh tim pengembang dalam membangun **Mail Service (Persuratan)**.

# Task Requirements (The "What" & "Why"):
1. **Resource Identification:** Identifikasi entitas pada modul Mail dan transformasikan dari tabel flat menjadi RESTful Resources yang logis.
2. **Mapping Dependencies:** Petakan data mana yang harus diambil dari database lokal (MariaDB) dan data mana yang harus dikonsumsi dari HR Service (API Kepegawaian).
3. **Security Design:** Rancang mekanisme validasi token AppWrite pada layer API Gateway/Security Filter Spring Boot.
4. **API Documentation Output:** Buat draft dokumentasi API yang mencakup:
    - **Endpoints:** (Contoh: GET /v1/mail/inbox, POST /v1/mail/disposition).
    - **HTTP Methods & Status Codes.**
    - **Request/Response Schema (JSON).**
    - **Query Parameters** untuk filtering, sorting, dan pagination.
    - **Error Handling** standards.

# Scope of Analysis:
- Fokus eksklusif pada modul Mail (Persuratan).
- Abaikan fungsi monolitik CI2 yang tidak relevan dengan persuratan.
- Pastikan desain mendukung Asynchronous processing jika ditemukan logic pengiriman notifikasi/email pada kode lama.

# Deliverables:
1. **API Contract Document** (Markdown format) yang mendetail untuk Mail Service.
2. **Entity-Relationship Mapping** antara database schema dan RESTful resources.
3. **Security Design Outline** untuk integrasi AppWrite token validation.
4. **Integration Points** dengan HR Service untuk data personil/struktur organisasi.

# Constraints:
- Harus mempertimbangkan backward compatibility dengan sistem lama selama fase transisi.
- Desain harus scalable untuk mendukung penambahan fitur di masa depan tanpa merombak API secara signifikan.
- Dokumentasi harus mudah dipahami oleh tim pengembang dengan berbagai tingkat pengalaman.

# Timeline:
- Analisis Database & CI2 Logic: 1 minggu
- Draft API Contract: 1 minggu
- Review & Finalization: 1 minggu
- Total: 3 minggu