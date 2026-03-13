# Phase 2: Core Mail Service — Business Logic Implementation

> **Goal:** Implementasi semua business logic untuk Mail Core  
> **Total Tasks:** 38 (28 P0, 10 P1)

---

## 2.1 Folder Management (Tasks 2.1 - 2.4)

### Task 2.1: GET /api/v1/mail/folders
**Priority:** P0 | **Estimate:** 20 min

**Description:** List mail folders for current user

**Acceptance Criteria:**
- [ ] Controller method with @GetMapping
- [ ] Service method returns List<MailFolderResponse>
- [ ] Include system folders (INBOX, SENT, DRAFT)
- [ ] Include user-created folders

**Endpoint:**
```
GET /api/v1/mail/folders
Response: List<MailFolderResponse>
```

**Files:**
- `controller/MailFolderController.java`
- `service/MailFolderService.java`

---

### Task 2.2: POST /api/v1/mail/folders
**Priority:** P0 | **Estimate:** 20 min

**Description:** Create new folder

**Acceptance Criteria:**
- [ ] Controller with @Valid
- [ ] Service validates folder name uniqueness per user
- [ ] Cannot create folder with system names
- [ ] Returns created folder

**Endpoint:**
```
POST /api/v1/mail/folders
Body: { name: string }
```

---

### Task 2.3: PUT /api/v1/mail/folders/{id}
**Priority:** P0 | **Estimate:** 20 min

**Description:** Update folder name

**Acceptance Criteria:**
- [ ] Controller with @Valid
- [ ] Cannot update system folders
- [ ] Returns updated folder

**Endpoint:**
```
PUT /api/v1/mail/folders/{id}
Body: { name: string }
```

---

### Task 2.4: DELETE /api/v1/mail/folders/{id}
**Priority:** P0 | **Estimate:** 20 min

**Description:** Delete folder (non-system)

**Acceptance Criteria:**
- [ ] Cannot delete system folders (INBOX, SENT, DRAFT)
- [ ] Cascade to UserTask (move to INBOX)
- [ ] Returns success response

---

## 2.2 Read Mail Operations (Tasks 2.5 - 2.9)

### Task 2.5: GET /api/v1/mail/folders/{id}/mails
**Priority:** P0 | **Estimate:** 60 min

**Description:** Read mails in folder with pagination

**Acceptance Criteria:**
- [ ] Controller with pagination params (page, size, sort)
- [ ] Service with HR API integration (cached)
- [ ] Enrich with sender name, recipient summary
- [ ] Returns PagedResponse<MailSummaryResponse>

**Endpoint:**
```
GET /api/v1/mail/folders/{id}/mails?page=0&size=20&sort=date,desc
```

**Files:**
- `dto/response/MailSummaryResponse.java`

---

### Task 2.6: GET /api/v1/mails/{id}
**Priority:** P0 | **Estimate:** 45 min

**Description:** Get mail detail

**Acceptance Criteria:**
- [ ] Full aggregation: mail + recipients + attachments
- [ ] HR data enrichment (sender position)
- [ ] Returns MailDetailResponse

**Files:**
- `dto/response/MailDetailResponse.java`
- `dto/response/RecipientResponse.java`

---

### Task 2.7: GET /api/v1/mail/counters
**Priority:** P0 | **Estimate:** 30 min

**Description:** Get unread count per folder

**Acceptance Criteria:**
- [ ] Returns Map<folderId, count>
- [ ] Include system folders
- [ ] Efficient COUNT query

**Endpoint:**
```
GET /api/v1/mail/counters
Response: { "INBOX": 5, "SENT": 0, "DRAFT": 2, ... }
```

---

### Task 2.8: PATCH /api/v1/mails/{id}/read
**Priority:** P0 | **Estimate:** 20 min

**Description:** Mark mail as read

**Acceptance Criteria:**
- [ ] Update UserTask read_status = true
- [ ] Idempotent (can call multiple times)
- [ ] Returns success

---

### Task 2.9: GET /api/v1/mails/{id}/read-status
**Priority:** P1 | **Estimate:** 30 min

**Description:** Get who has read the mail

**Acceptance Criteria:**
- [ ] List all recipients with read status
- [ ] Returns List<ReadStatusResponse>

**Files:**
- `dto/response/ReadStatusResponse.java`

---

## 2.3 Create/Update Mail Draft (Tasks 2.10 - 2.12)

### Task 2.10: POST /api/v1/mails
**Priority:** P0 | **Estimate:** 45 min

**Description:** Create draft mail

**Acceptance Criteria:**
- [ ] Controller with @Valid
- [ ] Service maps MailCreateRequest to Mail entity
- [ ] Returns MailResponse with generated temp ID

**Files:**
- `dto/request/MailCreateRequest.java`
- `dto/response/MailResponse.java`

---

### Task 2.11: PUT /api/v1/mails/{id}
**Priority:** P0 | **Estimate:** 30 min

**Description:** Update draft mail

**Acceptance Criteria:**
- [ ] Only draft can be updated
- [ ] Returns updated MailResponse

---

### Task 2.12: Create MailNumberGeneratorService
**Priority:** P0 | **Estimate:** 60 min

**Description:** Service for auto-generating mail number

**Acceptance Criteria:**
- [ ] Strategy Pattern for CLIENT_CODE (BMS, SMD, BPN)
- [ ] Template-based: `#seq#/#org_code#-#type#/#MR#/#YYYY#-#m_cat#`
- [ ] Sequence management (atomic, unique)
- [ ] Roman numeral month converter

**Files:**
- `service/MailNumberGeneratorService.java`
- `service/strategy/MailNumberStrategy.java`
- `service/strategy/BmsMailNumberStrategy.java`
- `service/strategy/SmdMailNumberStrategy.java`

---

## 2.4 Send Mail — Event-Driven Orchestration (Tasks 2.13 - 2.22)

### Task 2.13: POST /api/v1/mails/{id}/send
**Priority:** P0 | **Estimate:** 30 min

**Description:** Send mail endpoint

**Acceptance Criteria:**
- [ ] Validate mail has recipients
- [ ] @Transactional
- [ ] Publish SendMailEvent
- [ ] Returns success

---

### Task 2.14: Implement SendMailEvent
**Priority:** P0 | **Estimate:** 20 min

**Description:** Spring event for send orchestration

**Acceptance Criteria:**
- [ ] SendMailEvent class with mailId
- [ ] Event publisher in MailService
- [ ] 10 side-effects as event listeners

**Files:**
- `event/SendMailEvent.java`

---

### Task 2.15: Listener - Update Mail Status
**Priority:** P0 | **Estimate:** 15 min

**Description:** @EventListener for status update

**Acceptance Criteria:**
- [ ] Update status to SENT
- [ ] Set m_created_date = now
- [ ] @TransactionalEventListener

---

### Task 2.16: Listener - Generate Mail Number
**Priority:** P0 | **Estimate:** 20 min

**Description:** @EventListener for numbering

**Acceptance Criteria:**
- [ ] Call MailNumberGeneratorService
- [ ] Handle DuplicateNumberException
- [ ] Retry logic if needed

---

### Task 2.17: Listener - Create Inbox Tasks
**Priority:** P0 | **Estimate:** 30 min

**Description:** @EventListener for recipient inbox

**Acceptance Criteria:**
- [ ] Batch insert UserTask for each recipient
- [ ] Folder = INBOX
- [ ] Set read_status = false

---

### Task 2.18: Listener - Update Sender Draft
**Priority:** P0 | **Estimate:** 20 min

**Description:** @EventListener for sender folder

**Acceptance Criteria:**
- [ ] Move from DRAFT to SENT folder
- [ ] Create UserTask for sender

---

### Task 2.19: Listener - Mark Parent as Read
**Priority:** P1 | **Estimate:** 20 min

**Description:** @EventListener for reply handling

**Acceptance Criteria:**
- [ ] If parentId exists, mark parent as READ
- [ ] Move parent to READ folder

---

### Task 2.20: Listener - Track Response Time
**Priority:** P1 | **Estimate:** 20 min

**Description:** @EventListener for response tracking

**Acceptance Criteria:**
- [ ] Calculate response time if reply
- [ ] Insert to mail_respontime table

---

### Task 2.21: Listener - Update Statistics
**Priority:** P1 | **Estimate:** 20 min

**Description:** @EventListener for monthly stats

**Acceptance Criteria:**
- [ ] Upsert mail_category_statistic
- [ ] Upsert mail_org_statistic

---

### Task 2.22: Listener - Send Email Notification
**Priority:** P1 | **Estimate:** 30 min

**Description:** @EventListener async SMTP

**Acceptance Criteria:**
- [ ] @Async annotation
- [ ] Queue to smtp_mail_log
- [ ] Call SmtpMailService

---

## 2.5 Recipient Management (Tasks 2.23 - 2.27)

### Task 2.23: GET /api/v1/mails/{id}/recipients
**Priority:** P0 | **Estimate:** 30 min

**Description:** List recipients

**Acceptance Criteria:**
- [ ] HR data enrichment (emp_name, pos_name)
- [ ] Returns List<RecipientResponse>

---

### Task 2.24: POST /api/v1/mails/{id}/recipients
**Priority:** P0 | **Estimate:** 30 min

**Description:** Add single recipient

**Acceptance Criteria:**
- [ ] HR API lookup by empId or nipam
- [ ] Validate circulation type
- [ ] Create MailRecipient

---

### Task 2.25: POST /api/v1/mails/{id}/recipients/batch
**Priority:** P0 | **Estimate:** 30 min

**Description:** Add multiple recipients

**Acceptance Criteria:**
- [ ] List validation
- [ ] Batch insert
- [ ] Returns created recipients

---

### Task 2.26: DELETE /api/v1/mails/{id}/recipients/{rid}
**Priority:** P0 | **Estimate:** 15 min

**Description:** Remove recipient

**Acceptance Criteria:**
- [ ] Soft delete or hard delete
- [ ] Cannot delete if mail already sent

---

### Task 2.27: POST /api/v1/mails/{id}/recipients/copy
**Priority:** P1 | **Estimate:** 30 min

**Description:** Copy recipients from another mail

**Acceptance Criteria:**
- [ ] Deep copy logic
- [ ] Returns copied recipients

---

## 2.6 Mail Operations (Tasks 2.28 - 2.33)

### Task 2.28: DELETE /api/v1/mails/{id}
**Priority:** P0 | **Estimate:** 20 min

**Description:** Delete/trash mail

**Acceptance Criteria:**
- [ ] Move to DELETED folder
- [ ] Create UserTask

---

### Task 2.29: PATCH /api/v1/mails/{id}/move
**Priority:** P0 | **Estimate:** 20 min

**Description:** Move mail to folder

**Acceptance Criteria:**
- [ ] Update UserTask folder_id
- [ ] Support PERSONAL folders (>10)

---

### Task 2.30: PATCH /api/v1/mails/{id}/restore
**Priority:** P0 | **Estimate:** 20 min

**Description:** Restore from trash

**Acceptance Criteria:**
- [ ] Restore to restore_folder_id
- [ ] Clear DELETED status

---

### Task 2.31: DELETE /api/v1/mail/trash
**Priority:** P1 | **Estimate:** 20 min

**Description:** Empty trash

**Acceptance Criteria:**
- [ ] Bulk delete UserTask for user
- [ ] Hard delete option

---

### Task 2.32: GET /api/v1/mails/search
**Priority:** P0 | **Estimate:** 45 min

**Description:** Search mails

**Acceptance Criteria:**
- [ ] Dynamic query: keyword, typeId, categoryId, date range
- [ ] Returns PagedResponse

---

### Task 2.33: GET /api/v1/mails/{id}/tracking
**Priority:** P1 | **Estimate:** 45 min

**Description:** Track mail thread

**Acceptance Criteria:**
- [ ] Build n-level tree
- [ ] Returns ThreadTreeResponse

**Files:**
- `dto/response/ThreadTreeResponse.java`

---

## 2.7 System Notifications (Tasks 2.34 - 2.38)

### Task 2.34: Create SystemMailService
**Priority:** P0 | **Estimate:** 60 min

**Description:** Service for automated notifications

**Acceptance Criteria:**
- [ ] Template pattern for 4 notification types
- [ ] Eliminate 80% duplication
- [ ] Common method: createSystemMail()

**Files:**
- `service/SystemMailService.java`

---

### Task 2.35: createPublicationMailNotif
**Priority:** P1 | **Estimate:** 30 min

**Description:** Broadcast to all employees

**Acceptance Criteria:**
- [ ] Called via @EventListener
- [ ] Uses SystemMailService
- [ ] Creates mail + UserTask for all active employees

---

### Task 2.36: createTanggunganMailNotif
**Priority:** P1 | **Estimate:** 20 min

**Description:** Notification for dependency change

**Acceptance Criteria:**
- [ ] Called via @EventListener
- [ ] Related employees only

---

### Task 2.37: createApprovalCutiNotif
**Priority:** P1 | **Estimate:** 20 min

**Description:** Notification for leave approval

**Acceptance Criteria:**
- [ ] Called via @EventListener
- [ ] Related employees only

---

### Task 2.38: createArchiveAccessNotif
**Priority:** P1 | **Estimate:** 20 min

**Description:** Notification for archive access grant

**Acceptance Criteria:**
- [ ] Called via @EventListener
- [ ] Granted users only

---

## Phase 2 Checklist

- [ ] 2.1 List Folders
- [ ] 2.2 Create Folder
- [ ] 2.3 Update Folder
- [ ] 2.4 Delete Folder
- [ ] 2.5 Read Folder Mails
- [ ] 2.6 Get Mail Detail
- [ ] 2.7 Get Counters
- [ ] 2.8 Mark as Read
- [ ] 2.9 Get Read Status (P1)
- [ ] 2.10 Create Draft
- [ ] 2.11 Update Draft
- [ ] 2.12 Mail Number Generator
- [ ] 2.13 Send Endpoint
- [ ] 2.14 SendMailEvent
- [ ] 2.15-2.22 Event Listeners (8 tasks)
- [ ] 2.23-2.27 Recipient Management (5 tasks)
- [ ] 2.28-2.33 Mail Operations (6 tasks)
- [ ] 2.34-2.38 System Notifications (5 tasks)

**Previous Phase:** [Phase 1: Domain & Database](./01-domain-database.md)  
**Next Phase:** [Phase 3: Archive Service](./03-archive-service.md)
