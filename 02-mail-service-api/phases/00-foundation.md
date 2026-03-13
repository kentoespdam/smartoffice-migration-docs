# Phase 0: Foundation — Setup & Configuration

> **Goal:** Siapkan Spring Boot project dengan struktur package yang benar  
> **Total Tasks:** 10 (7 P0, 3 P1)

---

## Task List

### Task 0.1: Create Spring Boot Project
**Priority:** P0 | **Estimate:** 30 min

**Description:** Initialize project dengan Spring Initializr

**Acceptance Criteria:**
- [ ] Spring Boot 4.0.+ (Java 25) Graalvm selected
- [ ] Dependencies: Web, JPA, Security, Validation, Lombok, MariaDB driver, OpenAPI, Cache, Spring Boot DevTools, Spring Boot Actuator, Spring WebFlux (untuk WebClient), Flyway, JOOQ, Spring Boot Starter Test
- [ ] Project structure: `id.perumdamts.mail`
- [ ] Build tool: Gradle

---

### Task 0.2: Setup Package Structure
**Priority:** P0 | **Estimate:** 15 min

**Description:** Create package structure sesuai architectural design

**Acceptance Criteria:**
- [ ] `config/` — Spring configuration
- [ ] `controller/` — REST Controllers
- [ ] `service/` — Business Logic
- [ ] `repository/` — Spring Data JPA
- [ ] `entities/` — JPA Entities
- [ ] `entities/enums/` — Entity enums
- [ ] `dto/request/` — Request DTOs
- [ ] `dto/response/` — Response DTOs
- [ ] `integration/hr/` — HR API client
- [ ] `integration/appwrite/` — Auth client
- [ ] `exception/` — Custom exceptions

---

### Task 0.3: Configure application.properties
**Priority:** P0 | **Estimate:** 15 min

**Description:** Setup database connection, JPA, logging

**Acceptance Criteria:**
- [ ] MariaDB connection configured (URL, username, password)
- [ ] JPA show-sql enabled for dev
- [ ] Logging level: `org.springframework` = INFO, `id.perumdamts.mail` = DEBUG
- [ ] Profiles: dev, prod

**Configuration:**
```yaml
spring:
  datasource:
    url: jdbc:mariadb://localhost:3306/mail
    username: mail
    password: secret
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: validate

logging:
  level:
    id.perumdamts.mail: DEBUG
```

---

### Task 0.4: Setup Spring Security Config
**Priority:** P0 | **Estimate:** 45 min

**Description:** Configure JWT validation with AppWrite

**Acceptance Criteria:**
- [ ] `SecurityConfig` class created
- [ ] JWT token validation filter configured
- [ ] Public endpoints: `/api/v1/public/**`, `/swagger-ui/**`, `/v3/api-docs/**`
- [ ] Protected endpoints: `/api/v1/**`
- [ ] CORS configured for frontend

**Files to Create:**
- `config/SecurityConfig.java`
- `filter/JwtAuthFilter.java`

---

### Task 0.5: Create Base Exception Classes
**Priority:** P0 | **Estimate:** 30 min

**Description:** Create custom exception hierarchy

**Acceptance Criteria:**
- [ ] `MailNotFoundException` — 404
- [ ] `DuplicateNumberException` — 409
- [ ] `UnauthorizedArchiveAccessException` — 403
- [ ] `GlobalExceptionHandler` — @RestControllerAdvice
- [ ] Standard error response format

**Files to Create:**
- `exception/MailNotFoundException.java`
- `exception/DuplicateNumberException.java`
- `exception/UnauthorizedArchiveAccessException.java`
- `exception/GlobalExceptionHandler.java`

---

### Task 0.6: Create Base DTO Classes
**Priority:** P0 | **Estimate:** 20 min

**Description:** Create common DTO patterns

**Acceptance Criteria:**
- [ ] `PagedResponse<T>` — Pagination wrapper
- [ ] `ApiResponse<T>` — Standard response
- [ ] `ErrorResponse` — Error format
- [ ] All use Lombok @Data, @Builder

**Files to Create:**
- `dto/response/PagedResponse.java`
- `dto/response/ApiResponse.java`
- `dto/response/ErrorResponse.java`

---

### Task 0.7: Setup OpenAPI/Swagger Config
**Priority:** P1 | **Estimate:** 20 min

**Description:** Configure OpenAPI documentation

**Acceptance Criteria:**
- [ ] Swagger UI accessible at `/swagger-ui.html`
- [ ] API docs at `/v3/api-docs`
- [ ] API info: title, version, description
- [ ] Security scheme for JWT

**Files to Create:**
- `config/OpenApiConfig.java`

---

### Task 0.8: Create Enums
**Priority:** P0 | **Estimate:** 20 min

**Description:** Create all domain enums

**Acceptance Criteria:**
- [ ] `MailStatus` — DRAFT, SENT, ARCHIVED, DELETED
- [ ] `FolderType` — INBOX, SENT, DRAFT, READ, DELETED, PERSONAL
- [ ] `CirculationType` — TEMBUSAN, DISPOSISI, LAINNYA
- [ ] `ArchiveStatus` — DRAFT, PUBLISHED

**Files to Create:**
- `domain/enums/MailStatus.java`
- `domain/enums/FolderType.java`
- `domain/enums/CirculationType.java`
- `domain/enums/ArchiveStatus.java`

---

### Task 0.9: Setup WebClient for HR API
**Priority:** P0 | **Estimate:** 30 min

**Description:** Configure HTTP client for Kepegawaian API

**Acceptance Criteria:**
- [ ] WebClient bean configured
- [ ] JWT interceptor added
- [ ] Base URL: `http://192.168.1.214:8080`
- [ ] Error handling for 4xx, 5xx

**Files to Create:**
- `config/WebClientConfig.java`
- `integration/hr/HrServiceClient.java`

---

### Task 0.10: Setup Cache Configuration
**Priority:** P1 | **Estimate:** 30 min

**Description:** Configure Caffeine/Redis cache

**Acceptance Criteria:**
- [ ] `CacheConfig` class created
- [ ] TTL configured: Employee (15min), Position (60min)
- [ ] Cache names: `employees`, `positions`
- [ ] @EnableCaching annotation

**Files to Create:**
- `config/CacheConfig.java`

---

## Phase 0 Checklist

- [ ] 0.1 Spring Boot Project
- [ ] 0.2 Package Structure
- [ ] 0.3 Application Properties
- [ ] 0.4 Security Config
- [ ] 0.5 Exception Classes
- [ ] 0.6 Base DTOs
- [ ] 0.7 OpenAPI Config (P1)
- [ ] 0.8 Enums
- [ ] 0.9 HR WebClient
- [ ] 0.10 Cache Config (P1)

**Next Phase:** [Phase 1: Domain & Database](./01-domain-database.md)
