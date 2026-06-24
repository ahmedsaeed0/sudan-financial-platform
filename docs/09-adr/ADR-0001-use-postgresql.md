# ADR-0001 — Use PostgreSQL

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

The platform will process payments, refunds, settlements, ledger entries, reconciliation records, merchant data, and operational audit logs.

The database must support strong consistency and reliable transactions.

Financial systems require correctness before convenience.

---

## Decision

Use PostgreSQL as the primary relational database.

---

## Alternatives Considered

### MySQL

Pros:

- common in Laravel ecosystem
- easy hosting availability
- familiar to many developers

Cons:

- PostgreSQL provides stronger advanced features for this platform direction
- PostgreSQL has better support for partial indexes, JSONB, advanced locking, and future analytical patterns

### MongoDB / Document Database

Pros:

- flexible schema
- fast for some document workflows

Cons:

- not ideal as the primary source for financial ledger consistency
- weaker fit for relational financial data
- harder to enforce relational constraints

### SQLite

Pros:

- simple local development

Cons:

- not suitable as production database for this platform

---

## Consequences

Positive:

- strong ACID behavior
- row-level locking
- good indexing capabilities
- JSONB support
- good future scaling path
- strong fit for ledger and reconciliation

Negative:

- requires stronger PostgreSQL knowledge
- operational tuning is important
- connection management must be handled carefully at scale

---

## Implementation Notes

Use PostgreSQL for:

- merchants
- payments
- refunds
- settlement
- ledger
- reconciliation
- audit logs

Redis may be used for queue/cache/rate limiting, but Redis is not the source of truth.

---

## Rule

If data affects money, state, audit, or reconciliation, PostgreSQL must be the source of truth.
