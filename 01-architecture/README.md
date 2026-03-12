# Analisis Arsitektur & Desain Migrasi: Mail Service (Persuratan)
## SmartOffice - PHP CodeIgniter 2 → Spring Boot REST API

---

## Daftar Dokumen

1. [Domain Analysis](./01-domain-analysis.md) — Entitas, business logic, dan coupling dengan HR
2. [Decomposition Strategy](./02-decomposition-strategy.md) — Strategi pemisahan service dan data consistency
3. [Architectural Design](./03-architectural-design.md) — Layered architecture, package structure, endpoint design
4. [UML Visualization](./04-uml-visualization.md) — Component, sequence, dan class diagram
5. [Migration Risks & Recommendations](./05-migration-risks.md) — Risiko dan rekomendasi migrasi

## Diagram Files

Semua diagram PlantUML tersedia di folder [`puml/`](./puml/):

| Diagram | File | Preview |
|---------|------|---------|
| Component - Service Decoupling | [puml/component-diagram-overview.puml](./puml/component-diagram-overview.puml) | [SVG](./diagrams/component-diagram-overview.svg) |
| Component - Internal Architecture | [puml/component-diagram-internal.puml](./puml/component-diagram-internal.puml) | [SVG](./diagrams/component-diagram-internal.svg) |
| Sequence - Send Mail | [puml/sequence-send-mail.puml](./puml/sequence-send-mail.puml) | [SVG](./diagrams/sequence-send-mail.svg) |
| Sequence - Archive Mail | [puml/sequence-archive-mail.puml](./puml/sequence-archive-mail.puml) | [SVG](./diagrams/sequence-archive-mail.svg) |
| Sequence - Read Inbox | [puml/sequence-read-inbox.puml](./puml/sequence-read-inbox.puml) | [SVG](./diagrams/sequence-read-inbox.svg) |
| Class - Domain Entities | [puml/class-diagram-domain.puml](./puml/class-diagram-domain.puml) | [SVG](./diagrams/class-diagram-domain.svg) |

---

*Document generated from analysis of SmartOffice legacy codebase*
*Schema: smartoffice.sql (MariaDB 11.1.5)*
*Source: PHP CodeIgniter 2 RPC Model → Target: Spring Boot REST API*
