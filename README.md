# Sudan Financial Platform

Status: Sprint 0 — Documentation & Architecture  
Project Type: Financial Infrastructure Platform  
Initial Market: Sudan  
Initial Currency: SDG  
Primary Framework Direction: Laravel Modular Monolith

---

## Vision

Build the financial infrastructure layer for digital commerce in Sudan.

This project is not only a payment gateway.

It is intended to become a financial operating platform that supports:

- payment acceptance
- hosted checkout
- payment links
- merchant dashboard
- developer APIs
- refunds
- settlement
- reconciliation
- webhooks
- risk controls
- reporting
- ledger-based financial truth

---

## Current Stage

The project is currently in Sprint 0.

Sprint 0 means:

- no production code yet
- no migrations yet
- no controllers yet
- no models yet

The goal is to define the business, domain, architecture, and database before implementation.

---

## Core Engineering Principles

1. Documentation leads implementation.
2. Ledger is the financial source of truth.
3. Payments are business records, not balance records.
4. PostgreSQL is the source of truth.
5. Redis is not used as financial truth.
6. UUIDv7 is used for primary identifiers.
7. Money is stored as integer minor units.
8. Public APIs require idempotency where money can be affected.
9. Provider integration is abstracted behind contracts.
10. Critical events use durable delivery patterns.
11. Financial records are not hard deleted.
12. Financial lifecycles use state machines.

---

## Documentation

Main documentation lives in:

```text
docs/
```

Current sections:

```text
docs/
├── README.md
├── 02-domain/
├── 04-database/
└── 09-adr/
```

---

## Domain Documentation

See:

```text
docs/02-domain/
```

Includes:

- Merchant Domain
- Payment Domain
- Ledger Domain
- Refund Domain
- Settlement Domain
- Webhook Domain
- Risk Domain
- Provider Domain

---

## Database Documentation

See:

```text
docs/04-database/
```

Includes:

- database design
- table catalog
- relationships
- indexes and constraints
- migration order
- ledger schema
- money model
- idempotency storage
- retention and audit
- partitioning strategy

---

## Architecture Decision Records

See:

```text
docs/09-adr/
```

Initial ADRs:

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

## Implementation Gate

Before implementation begins, each module should have:

- domain document
- database tables documented
- lifecycle states defined
- audit requirements defined
- tests planned
- relevant ADRs accepted

---

## Project Rule

Do not write migrations before database design is reviewed.

Do not write financial logic before ledger rules are clear.

Do not integrate directly with one provider without Provider Abstraction.

Do not calculate merchant balance from payments table.
