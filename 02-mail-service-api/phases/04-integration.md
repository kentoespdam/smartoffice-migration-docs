# Phase 4: Integration — External Services

> **Goal:** Integrasi dengan HR API, AppWrite Auth, dan Notification services  
> **Total Tasks:** 10 (6 P0, 4 P1)

---

## 4.1 HR Service Integration (Tasks 4.1 - 4.4)

### Task 4.1: Create HrServiceClient
**Priority:** P0 | **Estimate:** 45 min

**Description:** Feign/WebClient for HR API

**Acceptance Criteria:**
- [ ] GET /pegawai/{id} — detail by ID
- [ ] GET /pegawai/{nipam}/nipam — search by NIPAM
- [ ] GET /pegawai/list — all employees (no pagination)
- [ ] GET /master/jabatan/{id} — position detail
- [ ] Error handling: 404 → empty, 5xx → retry/fallback

**Files:**
- `integration/hr/HrServiceClient.java`
- `integration/hr/HrApiConfig.java`

**Methods:**
```java
EmployeeResponse getEmployeeById(Long id);
EmployeeResponse getEmployeeByNipam(String nipam);
List<EmployeeResponse> getAllEmployees();
PositionResponse getPositionById(Long id);
```

---

### Task 4.2: Create HR DTOs
**Priority:** P0 | **Estimate:** 30 min

**Description:** DTOs for HR API responses

**Acceptance Criteria:**
- [ ] EmployeeResponse — basic info
- [ ] EmployeeProfileResponse — full profile
- [ ] PositionResponse — jabatan info
- [ ] OrganizationResponse — extracted from field

**Files:**
- `integration/hr/dto/EmployeeResponse.java`
- `integration/hr/dto/EmployeeProfileResponse.java`
- `integration/hr/dto/PositionResponse.java`

---

### Task 4.3: Implement Caching
**Priority:** P0 | **Estimate:** 45 min

**Description:** @Cacheable for HR calls

**Acceptance Criteria:**
- [ ] Employee cache: 15 min TTL
- [ ] Position cache: 60 min TTL
- [ ] Cache key strategy: `employees::{id}`, `positions::{id}`
- [ ] Cache eviction on update

**Files:**
- `integration/hr/CachedHrServiceClient.java`

**Annotations:**
```java
@Cacheable(value = "employees", key = "#id", unless = "#result == null")
@CacheEvict(value = "employees", key = "#id")
```

---

### Task 4.4: Create Background Sync Job
**Priority:** P1 | **Estimate:** 45 min

**Description:** Scheduled job for employee data

**Acceptance Criteria:**
- [ ] @Scheduled cron job (e.g., daily at 2 AM)
- [ ] Pre-load active employees
- [ ] Update cache
- [ ] Log sync stats

**Files:**
- `integration/hr/EmployeeSyncJob.java`

---

## 4.2 AppWrite Integration (Tasks 4.5 - 4.6)

### Task 4.5: Create AppWriteAuthClient
**Priority:** P0 | **Estimate:** 45 min

**Description:** Client for AppWrite API

**Acceptance Criteria:**
- [ ] JWT validation
- [ ] User session lookup
- [ ] Token refresh
- [ ] Logout endpoint

**Files:**
- `integration/appwrite/AppWriteAuthClient.java`
- `integration/appwrite/AppWriteConfig.java`

---

### Task 4.6: Implement User Context
**Priority:** P0 | **Estimate:** 30 min

**Description:** Extract user_id, emp_id, pos_id from JWT

**Acceptance Criteria:**
- [ ] UserContext class with ThreadLocal
- [ ] getCurrentUserId()
- [ ] getCurrentEmployeeId()
- [ ] getCurrentPositionId()
- [ ] Integration with SecurityContext

**Files:**
- `config/UserContext.java`
- `filter/JwtAuthFilter.java` (update)

---

## 4.3 Notification Services (Tasks 4.7 - 4.8)

### Task 4.7: Create SmtpMailService
**Priority:** P1 | **Estimate:** 45 min

**Description:** SMTP email sender

**Acceptance Criteria:**
- [ ] @Async method
- [ ] Template-based email (HTML)
- [ ] Queue to smtp_mail_log
- [ ] Retry on failure
- [ ] Attachment support

**Files:**
- `integration/notification/SmtpMailService.java`
- `config/SmtpConfig.java`

---

### Task 4.8: Create FirebasePushService
**Priority:** P1 | **Estimate:** 60 min

**Description:** Firebase push notification

**Acceptance Criteria:**
- [ ] Firebase Admin SDK integration
- [ ] Send to specific users by token
- [ ] Fallback handling (token expired)
- [ ] Batch send support

**Files:**
- `integration/notification/FirebasePushService.java`
- `config/FirebaseConfig.java`

---

## 4.4 Attachment Handling (Tasks 4.9 - 4.10)

### Task 4.9: Create Attachment Entity & Repository
**Priority:** P0 | **Estimate:** 30 min

**Description:** JPA for attachments table

**Acceptance Criteria:**
- [ ] Polymorphic reference (ref_type, ref_id)
- [ ] File metadata: name, size, ext, path
- [ ] Upload info: uploaded_by, upload_date
- [ ] Repository with findByRefTypeAndRefId

**Files:**
- `domain/Attachment.java`
- `repository/AttachmentRepository.java`

---

### Task 4.10: Attachment CRUD Endpoints
**Priority:** P0 | **Estimate:** 60 min

**Description:** File upload/download/delete

**Acceptance Criteria:**
- [ ] POST /api/v1/attachments — upload (MultipartFile)
- [ ] GET /api/v1/attachments/{id} — metadata
- [ ] GET /api/v1/attachments/{id}/download — download file
- [ ] DELETE /api/v1/attachments/{id} — delete
- [ ] GET /api/v1/attachments?refType=&refId= — list by reference
- [ ] File storage: filesystem (Phase 1) or S3/MinIO (Phase 2)

**Files:**
- `controller/AttachmentController.java`
- `service/AttachmentService.java`

---

## Phase 4 Checklist

- [ ] 4.1 HR Service Client
- [ ] 4.2 HR DTOs
- [ ] 4.3 HR Caching
- [ ] 4.4 Background Sync Job (P1)
- [ ] 4.5 AppWrite Auth Client
- [ ] 4.6 User Context
- [ ] 4.7 SMTP Service (P1)
- [ ] 4.8 Firebase Push (P1)
- [ ] 4.9 Attachment Entity
- [ ] 4.10 Attachment Endpoints

**Previous Phase:** [Phase 3: Archive Service](./03-archive-service.md)  
**Next Phase:** [Phase 5: Testing & Documentation](./05-testing-docs.md)
