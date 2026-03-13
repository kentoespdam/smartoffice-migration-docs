# Phase 1: Domain & Database — Entities & Repositories

> **Goal:** Create JPA Entities dan Repository untuk semua tabel  
> **Total Tasks:** 15 (15 P0, 0 P1)

---

## 1.1 Mail Core Entities

### Task 1.1: Create Mail Entity
**Priority:** P0 | **Estimate:** 45 min

**Description:** JPA entity for `mail` table

**Acceptance Criteria:**
- [ ] All fields mapped from legacy schema
- [ ] Relationships: @OneToMany recipients, @OneToMany attachments, @ManyToOne category, @ManyToOne type
- [ ] @Table(name="mail")
- [ ] Denormalized fields: m_created_by_name
- [ ] Threading fields: m_root_id, m_parent_id

**Fields:**
```
id, number, date, subject, content, note,
m_created_by, m_created_by_name,
type_id, category_id,
max_response_date,
incoming_mail_number, incoming_mail_source, incoming_mail_date,
outgoing_destination, outgoing_recipient_name,
status, created_date
```

**Files to Create:**
- `domain/Mail.java`

---

### Task 1.2: Create MailRecipient Entity
**Priority:** P0 | **Estimate:** 30 min

**Description:** JPA entity for `mail_recipient` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] @ManyToOne to Mail
- [ ] Denormalized fields: emp_name, pos_name
- [ ] CirculationType enum
- [ ] Flags: email_flag, sms_flag

**Files to Create:**
- `domain/MailRecipient.java`

---

### Task 1.3: Create MailFolder Entity
**Priority:** P0 | **Estimate:** 20 min

**Description:** JPA entity for `mail_folder` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] @OneToMany to UserTask
- [ ] FolderType enum
- [ ] System folders flag

**Files to Create:**
- `domain/MailFolder.java`

---

### Task 1.4: Create UserTask Entity
**Priority:** P0 | **Estimate:** 30 min

**Description:** JPA entity for `sys_user_task` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] @ManyToOne to Mail and MailFolder
- [ ] Read status flags
- [ ] restore_folder_id field

**Files to Create:**
- `domain/UserTask.java`

---

### Task 1.5: Create MailRepository
**Priority:** P0 | **Estimate:** 45 min

**Description:** Spring Data JPA repository for Mail

**Acceptance Criteria:**
- [ ] Custom query: findByFolderId (JOIN with UserTask)
- [ ] Custom query: findByRootId (threading)
- [ ] Pagination support
- [ ] Search query with dynamic conditions

**Files to Create:**
- `repository/MailRepository.java`

**Methods:**
```java
Page<Mail> findByUserTaskUserIdAndUserTaskFolderId(Long userId, Long folderId, Pageable pageable);
List<Mail> findByRootIdOrderByCreatedDateAsc(Long rootId);
Page<Mail> search(String keyword, Long typeId, Long categoryId, Pageable pageable);
```

---

### Task 1.6: Create MailRecipientRepository
**Priority:** P0 | **Estimate:** 20 min

**Description:** Spring Data JPA repository

**Acceptance Criteria:**
- [ ] Custom query: findByMailId
- [ ] Batch insert method
- [ ] Delete by mailId

**Files to Create:**
- `repository/MailRecipientRepository.java`

---

### Task 1.7: Create MailFolderRepository
**Priority:** P0 | **Estimate:** 20 min

**Description:** Spring Data JPA repository

**Acceptance Criteria:**
- [ ] Custom query: findByUserId
- [ ] Delete cascade handling
- [ ] System folder protection

**Files to Create:**
- `repository/MailFolderRepository.java`

---

### Task 1.8: CreateUserTaskRepository
**Priority:** P0 | **Estimate:** 20 min

**Description:** Spring Data JPA repository

**Acceptance Criteria:**
- [ ] Custom query: findByUserIdAndFolderId
- [ ] Batch operations
- [ ] Count unread by user and folder

**Files to Create:**
- `repository/UserTaskRepository.java`

---

## 1.2 Archive Entities

### Task 1.9: Create MailArchive Entity
**Priority:** P0 | **Estimate:** 30 min

**Description:** JPA entity for `mail_archive` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] Physical location fields: building, floor, room, rack, tier, box
- [ ] @OneToMany access list
- [ ] Status field (DRAFT, PUBLISHED)
- [ ] Secret type field

**Files to Create:**
- `domain/MailArchive.java`

---

### Task 1.10: Create MailArchiveAccess Entity
**Priority:** P0 | **Estimate:** 20 min

**Description:** JPA entity for `mail_archive_access` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] @ManyToOne to MailArchive
- [ ] Access flags: read, download, history
- [ ] pos_id reference

**Files to Create:**
- `domain/MailArchiveAccess.java`

---

### Task 1.11: Create MailArchiveRepository
**Priority:** P0 | **Estimate:** 30 min

**Description:** Spring Data JPA repository

**Acceptance Criteria:**
- [ ] Custom query: searchWithAccessControl (filter by pos_id)
- [ ] Custom query: findByOrgId
- [ ] Pagination support

**Files to Create:**
- `repository/MailArchiveRepository.java`

---

### Task 1.12: Create MailArchiveAccessRepository
**Priority:** P0 | **Estimate:** 15 min

**Description:** Spring Data JPA repository

**Acceptance Criteria:**
- [ ] Custom query: findByArchiveId
- [ ] Batch delete by archive
- [ ] Delete by pos_id

**Files to Create:**
- `repository/MailArchiveAccessRepository.java`

---

## 1.3 Master Data Entities

### Task 1.13: Create MailCategory Entity
**Priority:** P0 | **Estimate:** 15 min

**Description:** JPA entity for `mail_category` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] @OneToMany to Mail
- [ ] Code field for classification

**Files to Create:**
- `domain/MailCategory.java`

---

### Task 1.14: Create MailType Entity
**Priority:** P0 | **Estimate:** 20 min

**Description:** JPA entity for `mail_type` table

**Acceptance Criteria:**
- [ ] All fields mapped
- [ ] Tree structure (parent_id)
- [ ] @OneToMany to Mail
- [ ] Type code (I/M/E/K)

**Files to Create:**
- `domain/MailType.java`

---

### Task 1.15: Create MasterDataRepositories
**Priority:** P0 | **Estimate:** 15 min

**Description:** Repositories for Category & Type

**Acceptance Criteria:**
- [ ] MailCategoryRepository with findAll
- [ ] MailTypeRepository with tree query
- [ ] MailTypeRepository: findByParentIdIsNull for root types

**Files to Create:**
- `repository/MailCategoryRepository.java`
- `repository/MailTypeRepository.java`

---

## Phase 1 Checklist

- [ ] 1.1 Mail Entity
- [ ] 1.2 MailRecipient Entity
- [ ] 1.3 MailFolder Entity
- [ ] 1.4 UserTask Entity
- [ ] 1.5 MailRepository
- [ ] 1.6 MailRecipientRepository
- [ ] 1.7 MailFolderRepository
- [ ] 1.8 UserTaskRepository
- [ ] 1.9 MailArchive Entity
- [ ] 1.10 MailArchiveAccess Entity
- [ ] 1.11 MailArchiveRepository
- [ ] 1.12 MailArchiveAccessRepository
- [ ] 1.13 MailCategory Entity
- [ ] 1.14 MailType Entity
- [ ] 1.15 MasterDataRepositories

**Previous Phase:** [Phase 0: Foundation](./00-foundation.md)  
**Next Phase:** [Phase 2: Core Mail Service](./02-core-mail-service.md)
