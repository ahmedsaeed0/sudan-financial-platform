# Database Documentation

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

This directory contains the official database design documentation for the Sudan Financial Platform.

These documents must be reviewed before writing migrations.

---

## Documents

| File | Purpose |
|---|---|
| [01-database-design.md](./01-database-design.md) | Database principles and high-level design |
| [02-table-catalog.md](./02-table-catalog.md) | MVP table catalog and core fields |
| [03-relationships.md](./03-relationships.md) | Entity relationships and ownership |
| [04-indexes-and-constraints.md](./04-indexes-and-constraints.md) | Indexing and constraint strategy |
| [05-migration-order.md](./05-migration-order.md) | Recommended migration order |
| [06-ledger-schema.md](./06-ledger-schema.md) | Ledger schema and financial posting model |
| [07-data-retention-and-audit.md](./07-data-retention-and-audit.md) | Retention, deletion, and audit rules |
| [08-money-model.md](./08-money-model.md) | Money representation and fee snapshot rules |
| [09-idempotency-storage.md](./09-idempotency-storage.md) | Idempotency storage and safety rules |
| [10-partitioning-strategy.md](./10-partitioning-strategy.md) | Future partitioning and scaling strategy |

---

## Core Rules

1. PostgreSQL is the primary database.
2. UUIDv7 is used for primary identifiers.
3. Money is stored as integer minor units.
4. Currency is stored separately.
5. Ledger is the financial source of truth.
6. Payments are business records, not balance records.
7. Financial records are not hard deleted.
8. Sensitive actions are audited.
9. Idempotency must be enforced by database uniqueness.
10. Migrations are implementation of approved design, not the design itself.

---

## Review Checklist Before Migrations

Before creating any migration, confirm:

- Table purpose is clear.
- Domain ownership is clear.
- Relationships are documented.
- Indexes are justified.
- Unique constraints are defined.
- Sensitive fields are classified.
- Lifecycle/status values are known.
- Audit requirements are known.
- Ledger impact is known if money is involved.

---

## Implementation Rule

No database migration should be written until the related table is approved in this documentation.
