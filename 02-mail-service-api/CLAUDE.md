# AI Agent Guidelines — SmartOffice Mail Service Migration

**Context:** CodeIgniter 2.1.3 Ext.Direct RPC → Spring Boot REST API  
**Module:** Persuratan (Mail) | **Tasks:** 95 (72 P0, 23 P1)

---

## Core Rules

1. **One task = One session** — Each task fits one context window
2. **Phase order is strict** — Complete Phase N before N+1
3. **Priority order** — P0 (Critical) before P1 (Important)
4. **Verify acceptance criteria** — Each task has explicit criteria

## Workflow

```
1. Read task from phase file → 2. Check acceptance criteria → 
3. Implement → 4. Verify → 5. Commit → 6. Next task
```

---

## Phases

| # | File | Tasks | Focus |
|---|------|-------|-------|
| 0 | `phases/00-foundation.md` | 10 | Setup |
| 1 | `phases/01-domain-database.md` | 15 | Entities & Repositories |
| 2 | `phases/02-core-mail-service.md` | 38 | Business Logic |
| 3 | `phases/03-archive-service.md` | 12 | Archive & Access Control |
| 4 | `phases/04-integration.md` | 10 | External Integration |
| 5 | `phases/05-testing-docs.md` | 10 | Testing & Docs |

---

## Architecture

- **Pattern:** Layered (Controller → Service → Repository)
- **Design:** DDD, Repository pattern, DTO pattern
- **Testing:** JUnit 5 + Mockito, TestContainers

## Conventions

- Spring Boot best practices, Java 17+
- Clean code, self-documenting identifiers

---

## Task Checklist

**Before Start:**
- [ ] Task description read
- [ ] Acceptance criteria understood
- [ ] Dependencies verified (previous phase complete)
- [ ] Code structure reviewed

**Before Complete:**
- [ ] All acceptance criteria met
- [ ] Code compiles
- [ ] Tests pass
- [ ] Conventions followed
- [ ] Committed with clear message

---

## References

- [Implementation Plan](./IMPLEMENTATION_PLAN.md)
- [API Endpoints](./endpoints/)
- [Architecture](../01-architecture/)
