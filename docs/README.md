# Sudan Financial Platform — Documentation

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

This directory contains the official version-controlled documentation for the Sudan Financial Platform.

Notion is used for living planning and writing.

GitHub is used for approved or review-ready documentation.

---

## Documentation Structure

```text
docs/
├── 00-company/
├── 01-business/
├── 02-domain/
├── 03-architecture/
├── 04-database/
├── 05-api/
├── 06-security/
├── 07-devops/
├── 08-testing/
├── 09-adr/
└── 10-roadmap/
```

---

## Current Completed Sections

### Database

See:

```text
docs/04-database/
```

Contains:

- database principles
- table catalog
- relationships
- indexes and constraints
- migration order
- ledger schema
- data retention and audit
- money model
- idempotency storage
- partitioning strategy

### Architecture Decision Records

See:

```text
docs/09-adr/
```

Contains accepted decisions for:

- PostgreSQL
- UUIDv7
- Modular Monolith
- Double Entry Ledger
- Provider Abstraction
- Idempotency Keys
- Outbox Pattern
- State Machines
- No Hard Delete for Financial Records

---

## Core Engineering Rules

1. Documentation leads implementation.
2. No migration before database design approval.
3. Ledger is financial source of truth.
4. Payments are business records, not balance records.
5. Use UUIDv7 everywhere.
6. Use PostgreSQL as source of truth.
7. Use Provider Abstraction.
8. Use Idempotency-Key for payment creation.
9. Use Outbox Pattern for critical events.
10. Use State Machines for lifecycle entities.
11. Do not hard delete financial records.

---

## Implementation Gate

Before writing code for any module, confirm:

- domain document exists
- database tables are documented
- lifecycle/state machine is documented
- audit requirements are known
- tests are planned
- ADR exists for major architectural decisions

---

## Project Philosophy

This is not just a Laravel application.

This is a financial infrastructure platform.

Laravel is the first implementation framework, not the business model.
