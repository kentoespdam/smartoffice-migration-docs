# Phase 3: Archive Service — Archive & Access Control

> **Goal:** Implementasi Mail Archive dengan access control  
> **Total Tasks:** 12 (9 P0, 3 P1)

---

## 3.1 Archive CRUD Operations (Tasks 3.1 - 3.6)

### Task 3.1: GET /api/v1/archives
**Priority:** P0 | **Estimate:** 45 min

**Description:** List archives with pagination

**Acceptance Criteria:**
- [ ] Controller with pagination params
- [ ] Service with access control filter (by pos_id)
- [ ] Returns PagedResponse<ArchiveResponse>
- [ ] Only show archives user has access to

**Endpoint:**
```
GET /api/v1/archives?page=0&size=20
```

**Files:**
- `controller/MailArchiveController.java`
- `service/MailArchiveService.java`
- `dto/response/ArchiveResponse.java`

---

### Task 3.2: GET /api/v1/archives/search
**Priority:** P0 | **Estimate:** 45 min

**Description:** Search archives

**Acceptance Criteria:**
- [ ] Search params: keyword, date range, orgId
- [ ] Filter by pos_id access
- [ ] Returns PagedResponse

**Endpoint:**
```
GET /api/v1/archives/search?keyword=xxx&from=yyyy-MM-dd&to=yyyy-MM-dd
```

---

### Task 3.3: POST /api/v1/archives
**Priority:** P0 | **Estimate:** 45 min

**Description:** Create new archive

**Acceptance Criteria:**
- [ ] Controller with @Valid
- [ ] Service with ArchiveCreateRequest
- [ ] Physical location validation
- [ ] Returns created ArchiveResponse

**Files:**
- `dto/request/ArchiveCreateRequest.java`

---

### Task 3.4: PUT /api/v1/archives/{id}
**Priority:** P0 | **Estimate:** 30 min

**Description:** Update archive

**Acceptance Criteria:**
- [ ] Cannot update published archive
- [ ] Returns updated archive

---

### Task 3.5: PATCH /api/v1/archives/{id}/publish
**Priority:** P0 | **Estimate:** 30 min

**Description:** Publish archive (status → 2)

**Acceptance Criteria:**
- [ ] Validate archive has access list
- [ ] Update status to PUBLISHED
- [ ] Trigger notification event

---

### Task 3.6: DELETE /api/v1/archives/{id}
**Priority:** P0 | **Estimate:** 20 min

**Description:** Delete archive

**Acceptance Criteria:**
- [ ] Access validation (only creator/admin)
- [ ] Cascade delete access list
- [ ] Returns success

---

## 3.2 Archive Access Control (Tasks 3.7 - 3.9)

### Task 3.7: GET /api/v1/archives/{id}/access
**Priority:** P0 | **Estimate:** 30 min

**Description:** List access rights

**Acceptance Criteria:**
- [ ] Returns List<ArchiveAccessResponse>
- [ ] Enrich with position data from HR API

**Files:**
- `dto/response/ArchiveAccessResponse.java`

---

### Task 3.8: PUT /api/v1/archives/{id}/access
**Priority:** P0 | **Estimate:** 45 min

**Description:** Set access rights

**Acceptance Criteria:**
- [ ] Delete-and-reinsert pattern
- [ ] List of pos_id with flags
- [ ] Returns updated access list

**Endpoint:**
```
PUT /api/v1/archives/{id}/access
Body: [{ posId: 1, read: true, download: false, history: true }, ...]
```

---

### Task 3.9: Create ArchiveAccessService
**Priority:** P0 | **Estimate:** 45 min

**Description:** Service for access control logic

**Acceptance Criteria:**
- [ ] get_allowed_user() via HR API
- [ ] hasAccess() validation method
- [ ] getEmployeesByPositionId() helper

**Files:**
- `service/ArchiveAccessService.java`

---

## 3.3 Direct Archive from Mail (Tasks 3.10 - 3.12)

### Task 3.10: POST /api/v1/mails/{id}/archive
**Priority:** P0 | **Estimate:** 60 min

**Description:** Direct archive from mail

**Acceptance Criteria:**
- [ ] 3-gate validation:
  - User has archive menu permission
  - Mail has attachment(s)
  - Mail not already archived
- [ ] Copy attachments (REF_TYPE_MAIL → REF_TYPE_MAIL_ARCHIVE)
- [ ] Create MailArchive
- [ ] Returns archive response

**Endpoint:**
```
POST /api/v1/mails/{id}/archive
```

---

### Task 3.11: Create ArchiveNumberGenerator
**Priority:** P0 | **Estimate:** 45 min

**Description:** Service for archive numbering

**Acceptance Criteria:**
- [ ] Strategy Pattern for office (PUSAT vs cabang)
- [ ] Strategy Pattern for CLIENT_CODE (BMS/SMD/BPN)
- [ ] Sequence per office + type + year
- [ ] Template-based generation

**Files:**
- `service/ArchiveNumberGeneratorService.java`
- `service/strategy/ArchiveNumberStrategy.java`

---

### Task 3.12: Archive Notification Workflow
**Priority:** P1 | **Estimate:** 45 min

**Description:** Notif for archive access grant

**Acceptance Criteria:**
- [ ] get_unnotified() logic
- [ ] get_allowed_user() via HR API
- [ ] set_user_notif() batch insert
- [ ] Called on publish or access grant

**Files:**
- `service/MailArchiveNotificationService.java`

---

## Phase 3 Checklist

- [ ] 3.1 List Archives
- [ ] 3.2 Search Archives
- [ ] 3.3 Create Archive
- [ ] 3.4 Update Archive
- [ ] 3.5 Publish Archive
- [ ] 3.6 Delete Archive
- [ ] 3.7 Get Access List
- [ ] 3.8 Set Access Rights
- [ ] 3.9 ArchiveAccessService
- [ ] 3.10 Direct Archive from Mail
- [ ] 3.11 Archive Number Generator
- [ ] 3.12 Archive Notification (P1)

**Previous Phase:** [Phase 2: Core Mail Service](./02-core-mail-service.md)  
**Next Phase:** [Phase 4: Integration](./04-integration.md)
