# 🔍 Non-Trivial Business Logic — Modul Persuratan

## 📊 Quick Reference

| Function | Complexity | Priority | Pattern | Key Challenge |
|----------|-----------|----------|---------|----------------|
| `send()` | 🔴 Very High | Must refactor | Orchestration | 10 side-effects, needs event-driven |
| `generate_code()` | 🟠 High | Strategy | Strategy Pattern | Multi-tenant/client branching |
| `create_*_notif()` | 🟠 High | DRY | Template Pattern | 80% code duplication |
| Archive `generate_code()` | 🟠 High | Strategy | Strategy Pattern | Office + client branching |
| `directArsip()` | 🟡 Medium | Validation | Chain Pattern | Permission + business rules |
| Archive Access Control | 🟡 Medium | Security | ACL | Position-based permissions |
| `make_nlevel_threaded()` | 🟡 Medium | Optimize | CTE / In-Memory | Tree building |
| `signMe/checkSign` | 🟢 Low | Security | Enhance | Currently just `uniqid` |

---

## 1️⃣ 📧 send() — Pengiriman Surat

**File:** `mailmodel.php:812-1018` | **Complexity:** 🔴 Very High

Ini bukan sekadar update status—ada 10 side-effects dalam satu transaksi yang perlu di-orchestrate:

<details>
<summary><strong>Expand: 10-Step Side-Effects</strong></summary>

| Step | Logic | Spring Boot Equivalent |
|------|-------|------------------------|
| 1 | Validasi recipient (harus ada penerima) | `@PreCondition` / service validation |
| 2 | Auto-generate nomor surat | `MailCodeGeneratorService` |
| 3 | Update status → `SENT` + `m_created_date` | JPA update |
| 4 | Email notification queue (SMTP config + template) | `MailNotificationService` + `@Async` |
| 5 | Create inbox per recipient → `sys_user_task` | Event-driven batch insert |
| 6 | Update monthly statistics | `MailStatisticService` (upsert) |
| 7 | Move sender draft → sent | JPA update |
| 8 | Mark parent mail as READ (if reply) | JPA update |
| 9 | Response time tracking (if reply) | `MailResponseTimeService` |
| 10 | Trigger SMTP send (realtime) | `@Async` SMTP sender |

</details>

**Migration Note:** ⚠️ Refactor to event-driven orchestration using `@Transactional` + event listeners for decoupling (notifications, statistics, response tracking).

---

## 2️⃣ 🔢 generate_code() — Auto Numbering (Mail & Archive)

### Mail Numbering
**File:** `mailmodel.php:1564-1617` | **Complexity:** 🟠 High

Template-based nomor surat dengan multi-client branching:

```
Template: #seq#/1421002/#org_code#-#type#/#MR#/#YYYY#-#m_cat#
Example:  001/1421002/01-I/III/2026-000
```

| Placeholder | Source | Notes |
|-------------|--------|-------|
| `#seq#` | Auto-increment per prefix | Via `get_trans_sequence()` |
| `#org_code#` | User's organization code | From `mail_code` |
| `#type#` | Mail type (I/M/E/K) | Varies per `CLIENT_CODE` |
| `#MR#` | Roman month numeral | Helper `get_month_r()` |
| `#YYYY#` | Year | Current year |
| `#m_cat#` | Classification code | Mail category |

**Challenge:** `CLIENT_CODE` (BMS/SMD/BPN) punya template & type mapping berbeda  
**Refactor:** Strategy Pattern untuk client-specific code generation

### Archive Numbering
**File:** `mailarchivemodel.php` | **Complexity:** 🟠 High

Sama kompleks dengan mail numbering, plus:
- **Office-based branching:** PUSAT vs cabang → prefix berbeda
- **Sequence scope:** Per office → `prefix_sequence = office_code + type + year`  
- **Client variation:** Template berbeda per CLIENT_CODE (BMS/SMD/BPN)

---

## 3️⃣ 🌳 make_nlevel_threaded() + find_node() — Thread Tree Builder

**File:** `direct/mail.php:219-286` | **Complexity:** 🟡 Medium

Membangun n-level nested tree dari flat mail data untuk ExtJS TreePanel:

<details>
<summary><strong>Expand: Tree Building Logic</strong></summary>

**Data Structure:**
- `m_root_id` = root thread identifier
- `m_parent_id` = direct parent in thread (0 if top-level)

**Algorithm:**
1. Recursive `find_node()` searches for parent in current tree
2. Appends child to parent when found
3. **Fallback 1:** No parent found → insert at level 1 under root
4. **Fallback 2:** No root found → insert at top level

</details>

**Migration Path:** Keep in-memory tree building OR use recursive CTE query  
**Service:** `MailThreadService.buildTree()`
---

## 4️⃣ 📁 directArsip() — Direct Archive Workflow

**File:** `direct/mail.php:364-395` | **Complexity:** 🟡 Medium

Archive surat dengan validation chain:

<details>
<summary><strong>Expand: Validation Flow</strong></summary>

| Step | Validation | Action if Failed |
|------|------------|------------------|
| 1 | User has archive menu permission | Reject |
| 2 | Mail has attachment(s) | Reject |
| 3 | Mail not already archived | Reject (show who archived) |
| 4 | Copy attachments | REF_TYPE_MAIL → REF_TYPE_MAIL_ARCHIVE |

</details>

**Refactor:** Validation chain pattern + `@PreAuthorize`  
**Service:** `ArchiveService` with custom validators
---

## 5️⃣ ✍️ signMe() + checkSign() — Print Verification

**File:** `direct/mail.php:395-420` | **Complexity:** 🟢 Low

Currently a "print verification code" (not true digital signature):

<details>
<summary><strong>Expand: Current Implementation</strong></summary>

| Function | Logic |
|----------|-------|
| `signMe()` | Generate `uniqid()` → save to `print_log` (auth_code, date, mail_id, username, IP) |
| `checkSign()` | Lookup `print_log` by auth_code → render verification view |

</details>

**Current Flow:** `GET /api/mails/verify-sign/{key}` (simple lookup)  
**Enhancement:** Upgrade with JWT/UUID + QR code for better security                 
---

## 6️⃣ 📮 create_*_notif() — System Mail Notifications

**File:** `mailmodel.php:337-750` | **Complexity:** 🟠 High

Four automated notification types generating internal surat (not email):

| Function | Trigger | Recipient | File Range |
|----------|---------|-----------|------------|
| `create_publication_mail_notif` | Publication created | All active employees | L337-500 |
| `create_upd_emp_tanggungan_mail_notif` | Tanggungan data changed | Related employees | L500-600 |
| `create_approval_cuti_notif` | Cuti approval | Related employees | L600-700 |
| `create_archive_mail_notif` | Archive created + access granted | Granted users | L700-750 |

<details>
<summary><strong>Expand: Common Pattern (80% duplicated)</strong></summary>

All four follow identical structure:
1. Generate nomor surat (per CLIENT_CODE)
2. Fill template from `msg_template`
3. Insert into mail table
4. Set `m_root_id = self` (new thread)
5. Create `sys_user_task` entries:
   - Sent folder for admin
   - Inbox folder for recipients

</details>

**Problem:** 80% code duplication  
**Refactor:** Extract to `SystemMailService` + template pattern, called via Spring Events (@EventListener)

---

### 7. 🗄️ Archive generate_code() — Nomor Arsip (Terpisah dari Nomor Surat)

File: mailarchivemodel.php (bottom)

Sama kompleksnya dengan mail generate_code() tapi dengan:
- Office-based branching (PUSAT vs cabang → prefix berbeda)
- Sequence per office → prefix_sequence = office_code + type + year
- Template berbeda per CLIENT_CODE (BMS/SMD/BPN)

---

## 7️⃣ 🔐 Archive Access Control

**File:** `mailarchivemodel.php` | **Complexity:** 🟡 Medium

Position-based permission management with notification workflow:

<details>
<summary><strong>Expand: Access Control Flow</strong></summary>

**Permission Management:**
- `set_access()` → Delete-and-reinsert pattern per position
- 3 levels: access (read), download, history
- `search()` → Join `mail_archive_access` to filter by `pos_id` + flags

**Notification Workflow:**
```
set_notif() 
  → get_unnotified() 
  → get_allowed_user() 
  → cek_user_notif() 
  → set_user_notif()
```

</details>

**Refactor:** `ArchiveAccessService` + Spring Security @PreAuthorize  
**Notifications:** Scheduled job (`@Scheduled`) for batching

---

## 8️⃣ Summary & Refactor Roadmap

### Priority 1️⃣ — Critical Refactoring (Weeks 1-2)
- **`send()`** → Event-driven orchestration with @Transactional + async listeners
- **`create_*_notif()`** → Extract to `SystemMailService`, eliminate duplication

### Priority 2️⃣ — High Impact (Weeks 3-4)
- **`generate_code()` (mail & archive)** → Strategy Pattern for multi-client/office branching
- **Archive Access Control** → Spring Security @PreAuthorize + ACL

### Priority 3️⃣ — Medium Priority (Weeks 5-6)
- **`directArsip()`** → Validation chain with @PreAuthorize
- **`make_nlevel_threaded()`** → Test recursive CTE vs in-memory performance
- **`signMe/checkSign`** → Enhance with JWT/UUID + QR code

---

## Key Patterns to Use

| Pattern | Functions | Benefit |
|---------|-----------|---------|
| **Event-Driven** | `send()` | Decouple side-effects, async execution |
| **Strategy Pattern** | `generate_code()` | Eliminate multi-branch conditionals |
| **Template Pattern** | `create_*_notif()` | DRY, 80% code reuse |
| **Validation Chain** | `directArsip()` | Composable, testable rules |
| **Spring Security ACL** | Archive access | Declarative permission model |