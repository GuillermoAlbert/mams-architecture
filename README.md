# MAMS — Architecture Reference

<p>
  <img src="https://img.shields.io/badge/Spring_Boot-11203a?style=flat-square&logo=springboot&logoColor=white" alt="Spring Boot">
  <img src="https://img.shields.io/badge/Java_21-11203a?style=flat-square&logo=openjdk&logoColor=white" alt="Java 21">
  <img src="https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Liquibase-2563a8?style=flat-square&logo=liquibase&logoColor=white" alt="Liquibase">
  <img src="https://img.shields.io/badge/JWT-2563a8?style=flat-square&logo=jsonwebtokens&logoColor=white" alt="JWT">
</p>

**My Athlete Monitoring System (MAMS)** is a platform for training-load monitoring,
injury prevention, and athlete fitness tracking.

This repository documents the system architecture at a conceptual level. It contains
**no proprietary source code, internal class names, or confidential business logic** —
only the structural design, expressed as Mermaid diagrams that render natively on GitHub.

## Stack (at a glance)

- **API & domain:** Spring Boot 3.4 REST (Java 21)
- **Persistence:** JPA / Hibernate over PostgreSQL
- **Schema management:** Liquibase (versioned changesets)
- **Identity:** stateless JWT, role-based authorization
- **Frontend:** React 19 SPA (TypeScript)

> The architectural focus of this project is the **backend and data model**: multi-tenant
> authorization, an idempotent submission pipeline, and a domain schema designed around a
> single date axis. The diagrams below reflect that focus.

## Diagrams

| # | Document | What it shows |
|---|----------|---------------|
| 1 | [High-Level Architecture](./01-high-level-architecture.md) | Request flow from SPA through the API, service, and persistence layers to PostgreSQL. |
| 2 | [Permission Model](./02-permission-model.md) | Two-layer multi-tenant authorization: organization membership and team staffing. |
| 3 | [Domain Model (ERD)](./03-domain-model-erd.md) | Core domain entities and their key relationships. |
| 4 | [Data Submission Flow](./04-data-submission-flow.md) | How daily metrics are captured, persisted idempotently, and linked to sessions. |

> **Scope note:** MAMS ingests data from athlete-submitted forms and staff-managed
> training sessions. There is no external wearable/GPS vendor integration inside the
> application itself; that lives as a separate ETL service and is intentionally out of
> scope for this architecture reference.
