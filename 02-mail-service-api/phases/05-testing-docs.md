# Phase 5: Testing & Documentation — Quality Assurance

> **Goal:** Comprehensive testing dan documentation  
> **Total Tasks:** 10 (7 P0, 3 P1)

---

## 5.1 Unit Tests (Tasks 5.1 - 5.4)

### Task 5.1: Test MailService
**Priority:** P0 | **Estimate:** 60 min

**Description:** Unit tests for MailService

**Acceptance Criteria:**
- [ ] 80%+ coverage
- [ ] Mock repositories with Mockito
- [ ] Test all public methods
- [ ] Test exception scenarios

**Files to Create:**
- `src/test/java/.../service/MailServiceTest.java`

**Test Cases:**
```java
@Test void createDraft_success()
@Test void sendMail_withRecipients_success()
@Test void sendMail_noRecipients_throwsException()
@Test void markAsRead_success()
@Test void searchMails_withFilters_success()
```

---

### Task 5.2: Test MailNumberGenerator
**Priority:** P0 | **Estimate:** 45 min

**Description:** Unit tests for numbering

**Acceptance Criteria:**
- [ ] Test all CLIENT_CODE strategies (BMS, SMD, BPN)
- [ ] Test sequence uniqueness
- [ ] Test template formatting
- [ ] Test Roman numeral month converter

**Files to Create:**
- `src/test/java/.../service/MailNumberGeneratorServiceTest.java`

---

### Task 5.3: Test ArchiveService
**Priority:** P0 | **Estimate:** 45 min

**Description:** Unit tests for ArchiveService

**Acceptance Criteria:**
- [ ] Access control validation
- [ ] Publish workflow
- [ ] Search with filtering
- [ ] Exception scenarios

**Files to Create:**
- `src/test/java/.../service/MailArchiveServiceTest.java`

---

### Task 5.4: Test SystemMailService
**Priority:** P1 | **Estimate:** 45 min

**Description:** Unit tests for notifications

**Acceptance Criteria:**
- [ ] All 4 notification types
- [ ] Template rendering
- [ ] UserTask creation
- [ ] Batch processing

**Files to Create:**
- `src/test/java/.../service/SystemMailServiceTest.java`

---

## 5.2 Integration Tests (Tasks 5.5 - 5.7)

### Task 5.5: Test Mail Endpoints
**Priority:** P0 | **Estimate:** 90 min

**Description:** Integration tests for Mail API

**Acceptance Criteria:**
- [ ] @SpringBootTest
- [ ] Test all Mail endpoints (2.1 - 2.33)
- [ ] Verify database state after each test
- [ ] @Sql for test data setup
- [ ] @DirtiesContext for isolation

**Files to Create:**
- `src/test/java/.../controller/MailControllerIntegrationTest.java`

**Test Structure:**
```java
@SpringBootTest
@AutoConfigureMockMvc
@Testcontainers
class MailControllerIntegrationTest {
    @Autowired MockMvc mockMvc;
    @Autowired TestEntityManager entityManager;
    
    @Test void createAndSendMail()
    @Test void readFolder_withPagination()
    @Test void searchMails_withFilters()
}
```

---

### Task 5.6: Test Archive Endpoints
**Priority:** P0 | **Estimate:** 60 min

**Description:** Integration tests for Archive API

**Acceptance Criteria:**
- [ ] @SpringBootTest
- [ ] Test all Archive endpoints (3.1 - 3.12)
- [ ] Access control enforcement
- [ ] Publish workflow

**Files to Create:**
- `src/test/java/.../controller/MailArchiveControllerIntegrationTest.java`

---

### Task 5.7: Test HR Integration
**Priority:** P1 | **Estimate:** 45 min

**Description:** Integration tests with HR API mock

**Acceptance Criteria:**
- [ ] @MockBean for HrServiceClient
- [ ] Test caching behavior
- [ ] Test fallback handling
- [ ] WireMock for HTTP recording (optional)

**Files to Create:**
- `src/test/java/.../integration/hr/HrServiceClientTest.java`

---

## 5.3 Documentation (Tasks 5.8 - 5.10)

### Task 5.8: OpenAPI Documentation
**Priority:** P0 | **Estimate:** 60 min

**Description:** Complete API documentation

**Acceptance Criteria:**
- [ ] All endpoints documented with @Operation
- [ ] Request/Response examples with @ExampleObject
- [ ] Error codes documented
- [ ] Security scheme applied
- [ ] Swagger UI verified

**Annotations:**
```java
@Operation(summary = "Create draft mail", description = "...")
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "Success"),
    @ApiResponse(responseCode = "400", description = "Bad Request"),
    @ApiResponse(responseCode = "401", description = "Unauthorized")
})
```

---

### Task 5.9: README Update
**Priority:** P0 | **Estimate:** 30 min

**Description:** Project README with setup guide

**Acceptance Criteria:**
- [ ] Prerequisites (Java 17+, MariaDB, Maven/Gradle)
- [ ] Quick start guide (clone, config, run)
- [ ] Configuration reference (application.properties)
- [ ] API access instructions
- [ ] Troubleshooting section

**Files to Update:**
- `README.md`

---

### Task 5.10: Migration Checklist
**Priority:** P0 | **Estimate:** 30 min

**Description:** Checklist for production deployment

**Acceptance Criteria:**
- [ ] Database migration steps (Liquibase/Flyway)
- [ ] Configuration checklist (env vars, secrets)
- [ ] Rollback procedure
- [ ] Health check endpoints
- [ ] Monitoring setup (Actuator)

**Files to Create:**
- `DEPLOYMENT.md`
- `MIGRATION-CHECKLIST.md`

---

## Phase 5 Checklist

- [ ] 5.1 MailService Unit Tests
- [ ] 5.2 MailNumberGenerator Tests
- [ ] 5.3 ArchiveService Tests
- [ ] 5.4 SystemMailService Tests (P1)
- [ ] 5.5 Mail Endpoint Integration Tests
- [ ] 5.6 Archive Endpoint Integration Tests
- [ ] 5.7 HR Integration Tests (P1)
- [ ] 5.8 OpenAPI Documentation
- [ ] 5.9 README Update
- [ ] 5.10 Migration Checklist

**Previous Phase:** [Phase 4: Integration](./04-integration.md)

---

## 🎉 Completion Checklist

All phases completed:

- [ ] Phase 0: Foundation (10 tasks)
- [ ] Phase 1: Domain & Database (15 tasks)
- [ ] Phase 2: Core Mail Service (38 tasks)
- [ ] Phase 3: Archive Service (12 tasks)
- [ ] Phase 4: Integration (10 tasks)
- [ ] Phase 5: Testing & Documentation (10 tasks)

**Total: 95 tasks** ✅
