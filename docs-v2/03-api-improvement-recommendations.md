# Rekomendasi Perbaikan: API Kepegawaian untuk SmartOffice

**Tanggal:** 2026-03-12
**Prioritas:** HIGH - Blocking MailService optimization
**Konteks:** Analisis mismatch antara rencana awal dan implementasi aktual API Kepegawaian

---

## Executive Summary

API Kepegawaian v1.0.0 **SUDAH mendukung** query parameter filtering yang diperlukan untuk modul Persuratan SmartOffice:

✅ **Tersedia:**
- Filter by nama, NIPAM (autocomplete support)
- Filter by posisi/jabatan (`jabatanId`)
- Filter by unit kerja/organisasi (`organisasiId`)
- Filter by status pegawai (`statusPegawai`, `statusKerja`)
- Pagination support (`page`, `size`)

⚠️ **Gaps Yang Masih Ada:**
1. **Master data organisasi** - belum ada dedicated `/master/organisasi` endpoint (organisasi data hanya dalam pegawai object)
2. **Field selection** - belum support selective field return (always return full object)
3. **Dedicated search endpoint** - tidak ada `/pegawai/search` (tapi `/pegawai?nama=X` sudah cukup)

**Rekomendasi Aksi:**
1. **PRIORITY 1:** Create dedicated `/master/organisasi` endpoint untuk master organisasi
2. **PRIORITY 2:** Dokumentasi & wrapper endpoint di SmartOffice untuk autocomplete optimization
3. **PRIORITY 3:** Nice-to-have - field selection support di responses

---

## Masalah Utama vs. Rencana Awal

| Aspek | Rencana Awal | Implementasi Aktual | Impact |
|-------|--------------|-------------------|--------|
| **Filtering Pegawai** | `GET /pegawai?nama={keyword}&status=ACTIVE&posId={id}` | ✅ **SUDAH ADA** - `/pegawai?nama=X&statusPegawai=PEGAWAI&jabatanId=Y&organisasiId=Z` | ✅ **RESOLVED** - query params sudah tersedia |
| **Lookup Posisi** | `GET /pegawai/{posId}/position` | ✅ Dapat via `/pegawai?jabatanId={id}` | ✅ Partial - use existing endpoint |
| **Master Organisasi** | `GET /master/organisasi/{id}` (proposed) | ✅ Dapat via `/pegawai?organisasiId={id}` | ⚠️ Partial - workaround tersedia, dedicated endpoint pending |
| **Autocomplete** | `GET /pegawai/search?q={keyword}` | ✅ Gunakan `/pegawai?nama={keyword}&size=50` | ✅ **FUNCTIONAL** - query params sufficient |
| **Pagination** | Implicit via query params | ✅ `/pegawai` dengan `page`, `size`, `sortBy`, `sortDirection` | ✅ **FULL support** |

---

## Rekomendasi Perubahan API

### 1. **PRIORITY 1: Dokumentasi Query Parameters `/pegawai` (SUDAH TERSEDIA ✅)**

**Status:** Query parameters sudah diimplementasikan di API Kepegawaian v1.0.0

**Endpoint yang tersedia:**

```
GET /pegawai?nama={keyword}&statusPegawai={status}&jabatanId={id}&organisasiId={id}&page={n}&size={limit}
```

**Query Parameters yang tersedia:**

| Parameter | Type | Required | Catatan |
|-----------|------|----------|---------|
| `nama` | string | NO | Partial match, case-insensitive |
| `nipam` | string | NO | Lookup by NIPAM (exact match) |
| `nik` | string | NO | Lookup by NIK |
| `statusPegawai` | enum | NO | KONTRAK, CAPEG, PEGAWAI, CALON_HONORER, HONORER, NON_PEGAWAI |
| `jabatanId` | integer | NO | Filter by jabatan/posisi |
| `organisasiId` | integer | NO | Filter by unit kerja (organisasi) |
| `profesiId` | integer | NO | Filter by profesi |
| `golonganId` | integer | NO | Filter by golongan |
| `gradeId` | integer | NO | Filter by grade |
| `statusKerja` | enum | NO | KARYAWAN_AKTIF, BERHENTI_OR_KELUAR, DIRUMAHKAN, dll |
| `jenisKelamin` | enum | NO | LAKI_LAKI, PEREMPUAN |
| `page` | integer | NO | Page number (0-indexed) |
| `size` | integer | NO | Results per page (default behavior) |
| `sortBy` | string | NO | Field to sort by |
| `sortDirection` | enum | NO | asc atau desc |

**Contoh Request:**
```bash
# Autocomplete: cari pegawai by nama
curl -X GET "http://192.168.1.214:8080/pegawai?nama=John&statusPegawai=PEGAWAI&size=50&page=0" \
  -H "Authorization: Bearer <jwt-token>"

# Filter by organisasi
curl -X GET "http://192.168.1.214:8080/pegawai?organisasiId=1&statusPegawai=PEGAWAI&size=100" \
  -H "Authorization: Bearer <jwt-token>"

# Lookup by NIPAM
curl -X GET "http://192.168.1.214:8080/pegawai?nipam=12345678" \
  -H "Authorization: Bearer <jwt-token>"
```

**Expected Response:**
```json
{
  "status": "success",
  "data": [
    {
      "id": 1,
      "nama": "John Doe",
      "nipam": "12345678",
      "nik": "3101234567890123",
      "statusPegawai": "PEGAWAI",
      "statusKerja": "KARYAWAN_AKTIF",
      "jabatan": {
        "id": 1,
        "nama": "Manager"
      },
      "organisasi": {
        "id": 1,
        "nama": "IT Division"
      },
      "email": "john.doe@company.com"
    }
  ],
  "page": 0,
  "size": 50,
  "totalElements": 150,
  "totalPages": 3
}
```

**Benefit untuk Mail System:**
- ✅ Autocomplete dengan keyword → query `/pegawai?nama={keyword}&size=50`
- ✅ Pencarian penerima surat → direct query by NIPAM: `/pegawai?nipam={nipam}`
- ✅ Listing pegawai per divisi → `/pegawai?organisasiId={id}&statusPegawai=PEGAWAI`
- ✅ Performance optimization → API layer sudah provide filtering, cukup cache hasil

---

### 2. **PRIORITY 2: Optimasi Penggunaan `/pegawai` untuk Autocomplete (MINOR ENHANCEMENT)**

**Status:** Endpoint sudah support filtering, rekomendasi minor:

**Rekomendasi:**
Buat wrapper/dedicated endpoint di SmartOffice API saja (bukan di Kepegawaian API) untuk:
- Standardisasi response structure untuk mail UI
- Cache optimization per tenant/module
- Field selection (return hanya fields yang diperlukan UI)

**Design SmartOffice Wrapper Endpoint:**

```
GET /mail-api/recipients/search?q={keyword}&limit=20
```

**Implementation di Spring Boot MailService:**

```java
@RestController
@RequestMapping("/mail-api/recipients")
public class MailRecipientSearchController {

    @GetMapping("/search")
    public ResponseEntity<MailRecipientSearchResponse> search(
            @RequestParam String q,
            @RequestParam(defaultValue = "20") int limit) {

        // Call Kepegawaian API
        List<Pegawai> results = kepegawaianClient.search(
            nama = q,
            statusPegawai = "PEGAWAI",
            size = limit
        );

        // Map to MailRecipientSearchResponse (simplified)
        return ResponseEntity.ok(new MailRecipientSearchResponse(results));
    }
}
```

**Expected Response:**
```json
{
  "status": "success",
  "results": [
    {
      "id": 1,
      "nama": "John Doe",
      "nipam": "12345678",
      "email": "john.doe@company.com",
      "jabatan": "Manager",
      "organisasi": "IT Division"
    }
  ],
  "total": 5
}
```

**Benefit:**
- ✅ Konsistensi response structure di mail module
- ✅ Caching optimization per module
- ✅ Abstraksi dari Kepegawaian API changes
- ✅ Field selection & transformation

---

### 3. **PRIORITY 3: OPEN ITEM - Master Data Endpoints untuk Organisasi**

**Status:** Belum tersedia endpoint dedicated untuk organisasi. Saat ini organisasi data hanya dalam response pegawai object.

#### 3.1 **Rekomendasi: GET `/master/organisasi` - List Unit Kerja (PENDING)**

```
GET /master/organisasi?page=0&size=100&sortBy=nama
```

**Why needed:**
- Populate dropdown "pilih unit kerja" di mail form
- Cache master organisasi once pada startup
- Enable filtering pegawai per organisasi lebih efisien

**Proposed Response:**
```json
{
  "status": "success",
  "data": [
    {
      "id": 1,
      "nama": "IT Division",
      "kode": "IT",
      "parentId": 0,
      "level": 1,
      "active": true
    }
  ],
  "page": 0,
  "size": 100,
  "totalElements": 50,
  "totalPages": 1
}
```

**Current Workaround:**
- Extract organisasi data dari `/pegawai?organisasiId={id}` response
- Atau gunakan `/master/jabatan/organisasi/{id}` untuk get position hierarchy

#### 3.2 **Endpoint Existing untuk Organisasi: `/master/jabatan/organisasi/{id}`**

```
GET /master/jabatan/organisasi/{organisasiId}
```

**Purpose:** Get semua jabatan dalam organisasi tertentu

**Note:** Ini give position hierarchy tapi bukan organisasi master data langsung

---

### 4. **PRIORITY 4: Improve `/pegawai/{id}/profil` Response**

**Current Issue:** Response mungkin overloaded dengan data tidak perlu

**Recommendation:**
```
GET /pegawai/{id}/profil?fields=nama,email,position,organisasi,gaji
```

Allow client-side field selection untuk optimize response size.

---

## Impact Analysis untuk SmartOffice Mail Module

### Skenario 1: Recipient Autocomplete

**Status Sebelumnya (TANPA query param filtering):**
```
1. GET /pegawai/list → load 5000+ pegawai
2. Cache di Redis (full payload ~50MB)
3. Filter di Java (nama, posisi, org)
4. Return top 20 hasil
⏱️ Response time: 500ms-2s (cache hit)
💾 Memory: ~50MB+ per application instance
```

**Status Sekarang (DENGAN query param filtering di API):**
```
Option A - Direct API call:
1. GET /pegawai?nama=john&size=20
2. API filter & return 20 hasil langsung
3. Render di UI
⏱️ Response time: 100-300ms (no cache needed)

Option B - Dengan caching strategy (recommended):
1. Periodic sync: Background job setiap jam
   GET /pegawai?page=0&size=1000 → cache di Caffeine
2. Autocomplete: Query cache + filter in memory
⏱️ Response time: 10-50ms (cache hit)
💾 Memory: ~5-10MB (filtered cache, tidak full 50MB)
```

**Improvement: 80-90% latency reduction (if caching implemented), 90% memory reduction**

---

### Skenario 2: Send Mail Notification ke Penerima

**Saat ini:**
```
1. Get recipients dari mail_recipient table (menyimpan emp_id)
2. Query cache untuk resolve email/nama
3. If not in cache → call GET /pegawai/{id}/profil
⏱️ Risk: Cache miss → 500ms+ delay per recipient
```

**Dengan enhancement:**
```
1. Get recipients dari mail_recipient
2. Query cache (pre-warmed via background job)
3. Fallback: GET /pegawai/{id}/profil (faster API)
⏱️ Guaranteed: <100ms per recipient
```

---

## Implementation Plan untuk SmartOffice

### Phase A: Interim Solution (Tanpa Perlu Wait Kepegawaian API)

**Timeline:** 1-2 minggu

1. **Implement caching di SmartOffice:**
   ```java
   @Service
   public class KepegawaianCacheService {
       @Scheduled(fixedRate = 3600000) // 1 jam
       public void syncEmployeeCache() {
           List<Pegawai> allEmployees = restTemplate.getForObject(
               "{baseUrl}/pegawai/list",
               new ParameterizedTypeReference<List<Pegawai>>() {}
           );
           cacheManager.getCache("pegawai").putAll(allEmployees);
       }
   }
   ```

2. **Add in-memory filtering di MailService:**
   ```java
   public List<Pegawai> searchRecipients(String keyword) {
       return pegawaiCache.stream()
           .filter(p -> p.getNama().contains(keyword) || p.getNipam().contains(keyword))
           .filter(p -> p.getStatus().equals("ACTIVE"))
           .sorted(Comparator.comparing(Pegawai::getNama))
           .limit(50)
           .collect(Collectors.toList());
   }
   ```

3. **Update MailService.addRecipient():**
   - Validate NIPAM via cache
   - Resolve email/position from cache
   - Cache miss → fallback to API

---

### Phase B: API Enhancement (Koordinasi dengan Tim Kepegawaian)

**Timeline:** 3-4 minggu

1. **Request Kepegawaian team untuk implement:**
   - [ ] Add query parameters to `/pegawai` (PRIORITY 1)
   - [ ] Create `/pegawai/search` endpoint (PRIORITY 2)
   - [ ] Create `/master/organisasi` endpoint (PRIORITY 3)
   - [ ] Improve field selection in responses (PRIORITY 4)

2. **Testing & validation:**
   - [ ] Load test dengan 10,000+ pegawai
   - [ ] Latency test untuk autocomplete (target <200ms)
   - [ ] Update SmartOffice KepegawaianClient untuk use new endpoints

---

## Detailed Endpoint Specifications

### Endpoint 1: Enhanced GET `/pegawai`

**OpenAPI Specification:**

```yaml
/pegawai:
  get:
    tags:
      - Pegawai
    summary: List pegawai dengan filtering
    parameters:
      - name: nama
        in: query
        schema:
          type: string
        description: Partial match pegawai nama (case-insensitive)
        example: John

      - name: nipam
        in: query
        schema:
          type: string
        description: NIPAM (exact or partial match)
        example: "12345"

      - name: status
        in: query
        schema:
          type: string
          enum: [ACTIVE, INACTIVE, ON_LEAVE]
        description: Employee status filter
        example: ACTIVE

      - name: organisasiId
        in: query
        schema:
          type: integer
        description: Unit kerja ID (organisasi)
        example: 1

      - name: positionId
        in: query
        schema:
          type: string
        description: Jabatan ID
        example: POS-001

      - name: limit
        in: query
        schema:
          type: integer
          default: 50
          maximum: 500
        description: Hasil per halaman

      - name: offset
        in: query
        schema:
          type: integer
          default: 0
        description: Offset pagination

    responses:
      '200':
        description: Success
        content:
          application/json:
            schema:
              type: object
              properties:
                status:
                  type: string
                data:
                  type: array
                  items:
                    $ref: '#/components/schemas/Pegawai'
                pagination:
                  type: object
                  properties:
                    total:
                      type: integer
                    limit:
                      type: integer
                    offset:
                      type: integer
```

### Endpoint 2: New GET `/pegawai/search`

```yaml
/pegawai/search:
  get:
    tags:
      - Pegawai
    summary: Real-time search pegawai (untuk autocomplete)
    parameters:
      - name: q
        in: query
        required: true
        schema:
          type: string
        description: Keyword pencarian (nama atau NIPAM)
        example: john

      - name: limit
        in: query
        schema:
          type: integer
          default: 20
          maximum: 50

      - name: exclude_inactive
        in: query
        schema:
          type: boolean
          default: true
        description: Exclude pegawai dengan status nonaktif

    responses:
      '200':
        description: Success
        content:
          application/json:
            schema:
              type: object
              properties:
                status:
                  type: string
                results:
                  type: array
                  items:
                    type: object
                    properties:
                      id:
                        type: string
                      nama:
                        type: string
                      nipam:
                        type: string
                      email:
                        type: string
                      position:
                        type: string
                      organisasi:
                        type: string
                total:
                  type: integer
```

---

## Communication Template untuk Kepegawaian Team

**Email Subject:** API Enhancement Request - Kepegawaian API v1.1.0 (SmartOffice Integration)

**Body:**

```
Halo Tim Kepegawaian,

Kami sedang melakukan migrasi modul Persuratan (Mail System) ke Spring Boot REST API.
Dalam analisis, kami menemukan bahwa API Kepegawaian v1.0.0 saat ini kurang mendukung
query parameter filtering yang diperlukan untuk implementasi mail module secara optimal.

Kami menghargai desain API yang ada, namun untuk mendukung skalabilitas dan performance
modul Persuratan, kami mengusulkan enhancement berikut (dalam prioritas):

**PRIORITY 1 (High Impact):**
- ✅ SUDAH ADA: Query parameters ke GET /pegawai: ?nama, ?statusPegawai, ?organisasiId, ?jabatanId
- Ini akan enable client untuk filter data di API layer, bukan di aplikasi

**PRIORITY 2 (High Impact):**
- Buat dedicated endpoint GET /pegawai/search?q={keyword} untuk autocomplete use case
- Response lebih kecil (hanya nama, nipam, email, position)

**PRIORITY 3 (Medium Impact):**
- Buat GET /master/organisasi endpoint untuk master data organisasi
- Saat ini data organisasi hanya tersedia di field pegawai

**PRIORITY 4 (Nice to Have):**
- Implement field selection di profil endpoint: /pegawai/{id}/profil?fields=nama,email,position

Kami siap untuk:
- Kolaborasi dalam desain API specification
- Provide use case details dan testing scenario
- Testing dan validation setelah implementation

Detail lengkap ada di attachment [03-api-improvement-recommendations.md].

Terima kasih!

Regards,
SmartOffice Development Team
```

---

## Backup Implementation: SmartOffice Side (Jika API Tidak Bisa Update)

Jika Kepegawaian team tidak bisa update API dalam timeline, SmartOffice bisa implement:

### 1. **Aggressive Caching Strategy**

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new CaffeineCacheManager("pegawai", "organisasi", "position");
    }
}

@Service
public class KepegawaianClient {

    @Cacheable(value = "pegawai", cacheManager = "cacheManager")
    public List<Pegawai> getAllEmployees() {
        return restTemplate.getForObject(
            "{baseUrl}/pegawai/list",
            new ParameterizedTypeReference<List<Pegawai>>() {}
        );
    }

    @Scheduled(fixedRate = 3600000) // Refresh every 1 hour
    public void evictCache() {
        cacheManager.getCache("pegawai").clear();
    }
}
```

### 2. **In-Memory Full-Text Search**

```java
@Service
public class MailRecipientSearchService {

    private final KepegawaianClient kepegawaianClient;
    private final SearchIndexer searchIndexer;

    public List<Pegawai> search(String keyword) {
        List<Pegawai> allEmployees = kepegawaianClient.getAllEmployees();

        return allEmployees.stream()
            .filter(p -> matchesKeyword(p, keyword))
            .filter(p -> p.getStatus().equals("ACTIVE"))
            .sorted(Comparator.comparing(p -> calculateRelevance(p, keyword)).reversed())
            .limit(50)
            .collect(Collectors.toList());
    }

    private boolean matchesKeyword(Pegawai p, String keyword) {
        return p.getNama().toLowerCase().contains(keyword.toLowerCase()) ||
               p.getNipam().contains(keyword);
    }

    private int calculateRelevance(Pegawai p, String keyword) {
        if (p.getNama().equalsIgnoreCase(keyword)) return 1000;
        if (p.getNama().toLowerCase().startsWith(keyword.toLowerCase())) return 500;
        return 100;
    }
}
```

### 3. **Background Sync Job**

```java
@Component
public class KepegawaianSyncJob {

    @Scheduled(fixedRate = 3600000) // Every 1 hour
    public void syncEmployeeData() {
        log.info("Starting employee data sync...");

        List<Pegawai> employees = kepegawaianClient.getAllEmployees();

        // Update local cache/database
        employeeRepository.saveAll(employees);

        log.info("Synced {} employees", employees.size());
    }
}
```

---

## Summary & Next Steps

| Priority | Rekomendasi | Timeline | Owner |
|----------|-------------|----------|-------|
| **P1** | Add query params to `/pegawai` | 3-4 minggu | Kepegawaian team |
| **P2** | Create `/pegawai/search` endpoint | 3-4 minggu | Kepegawaian team |
| **P3** | Create `/master/organisasi` endpoint | 2-3 minggu | Kepegawaian team |
| **P4** | Field selection in responses | 1-2 minggu | Kepegawaian team |
| **Interim** | Implement client-side caching | 1-2 minggu | SmartOffice team |

---

## Document Metadata

- **Version:** 1.0.0
- **Date:** 2026-03-12
- **Author:** SmartOffice Development Team
- **Status:** Ready for Review
- **Audience:** Kepegawaian API Team, SmartOffice Architecture
- **Related Documents:**
  - [02-migration-goals.md](02-migration-goals.md)
  - API Docs: http://192.168.1.214:8080/v3/api-docs
