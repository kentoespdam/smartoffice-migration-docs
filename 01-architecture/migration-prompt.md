# ROLE
You are acting as a Senior Software Architect specializing in Microservices Migration and Spring Boot Ecosystem.

# CONTEXT
We are migrating a legacy "SmartOffice" application.
- **Current Stack:** PHP CodeIgniter 2 using an RPC (Remote Procedure Call) model.
- **Target Stack:** Spring Boot REST API.
- **Architecture Goal:** Decompose the existing Monolith into two decoupled services:
  1. **HR Service (Kepegawaian)** - Already exists as a REST API (Integration Point).
  2. **Mail Service (Persuratan)** - To be designed.

# OBJECTIVE
Analyze the existing PHP CodeIgniter 2 codebase and database schema to design a modern RESTful architecture for the **Mail Service**. Focus strictly on **WHAT** and **WHY** (Architecture & Design) rather than **HOW** (Implementation/Coding).

# INPUTS & SCOPE
- **Schema:** Refer to `smartoffice.sql`.
- **Data Access:** 
    - **MariaDB** (192.168.230.84:3307 database: smartoffice user:dev password:password).
    - **Api Kepegawaian:** http://192.168.1.214:8080/v3/api-docs
    - **Authentication:** AppWrite version 1.3.4 (self-hosted) for user management and authentication.
- **Focus Area:** Mail (Persuratan) module only.
- **Files to Analyze:** Controllers and Models within the "Mail" scope. You may reference other files only if they have direct dependencies on the Mail module.
- **Integration:** The Mail service must interact with the existing HR REST API for personnel data. and must use AppWrite for authentication.

# REQUIRED TASKS
1. **Domain Analysis:** Identify core entities and logic within the Mail module from the legacy CI2 RPC model.
2. **Decomposition Strategy:** Explain how to separate Mail logic from the HR monolith, including how data consistency is maintained (e.g., using IDs/DTOs from the HR REST API instead of direct DB joins).
3. **Architectural Design:** Define the new Spring Boot structure (Controller-Service-Repository layers). Justify the use of DTOs for data transfer between services and the choice of RESTful endpoints.
4. **UML Visualization:** Create PlantUML diagrams for:
   - **Component Diagram:** Showing the decoupling between HR Service and Mail Service (split file into separate components).
   - **Sequence Diagram:** Illustrating a core Mail process (e.g., sending/tracking mail) that involves fetching data from the HR REST API (http://192.168.1.214:8080/v3/api-docs), (Include authentication flow with AppWrite). (split file into separate components).
   - **Class Diagram:** Focus on the domain entities for the Mail Service, including relationships and key attributes (e.g., Mail, Attachment, Archive), including their associations and cardinalities (master data).

# CONSTRAINTS & OUTPUT FORMAT
- **Language:** Analysis in Indonesian and English if necessary.
- **UML:** Provide raw PlantUML code blocks.
- **No Implementation:** Do not generate Java/Spring Boot boilerplate unless it's necessary to explain a specific architectural pattern.
- **Justification:** Every design choice must include a "Why" (e.g., Why use a certain DTO pattern or why separate a specific table).