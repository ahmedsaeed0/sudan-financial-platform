# Architecture Decision Records

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

Architecture Decision Records document important engineering decisions and the reasoning behind them.

Each ADR should explain:

- Context
- Decision
- Alternatives considered
- Consequences
- Status

---

## ADR Index

| ADR | Decision | Status |
|---|---|---|
| [ADR-0001](./ADR-0001-use-postgresql.md) | Use PostgreSQL | Accepted |
| [ADR-0002](./ADR-0002-use-uuidv7.md) | Use UUIDv7 Everywhere | Accepted |
| [ADR-0003](./ADR-0003-use-modular-monolith.md) | Use Modular Monolith | Accepted |
| [ADR-0004](./ADR-0004-use-double-entry-ledger.md) | Use Double Entry Ledger | Accepted |
| [ADR-0005](./ADR-0005-use-provider-abstraction.md) | Use Provider Abstraction | Accepted |
| [ADR-0006](./ADR-0006-use-idempotency-keys.md) | Use Idempotency Keys | Accepted |
| [ADR-0007](./ADR-0007-use-outbox-pattern.md) | Use Outbox Pattern | Accepted |
| [ADR-0008](./ADR-0008-use-state-machines.md) | Use State Machines | Accepted |
| [ADR-0009](./ADR-0009-no-hard-delete-financial-records.md) | No Hard Delete for Financial Records | Accepted |

---

## Rule

If a technical decision affects money, data integrity, security, scaling, provider integration, or developer experience, it must have an ADR.
