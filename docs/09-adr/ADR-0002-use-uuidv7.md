# ADR-0002 — Use UUIDv7 Everywhere

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

The platform will contain many entities that must be globally unique and safe to expose through APIs.

Examples:

- merchants
- payments
- refunds
- settlements
- webhook deliveries
- journal entries
- API keys

Sequential integer IDs are simple but expose internal sequence and can become awkward when systems are split or merged.

UUIDv4 is globally unique but random, which can create poor index locality.

---

## Decision

Use UUIDv7 for primary identifiers across the platform.

---

## Alternatives Considered

### Auto-increment integers

Pros:

- simple
- compact
- fast

Cons:

- exposes internal growth
- harder in distributed systems
- less suitable for public references

### UUIDv4

Pros:

- globally unique
- widely supported

Cons:

- random ordering
- weaker index locality

### ULID

Pros:

- sortable
- developer friendly

Cons:

- UUIDv7 is becoming a stronger standard option for time-ordered UUIDs

---

## Consequences

Positive:

- globally unique IDs
- time-sortable identifiers
- safer public references
- better index locality than UUIDv4
- good long-term scaling fit

Negative:

- slightly larger than integers
- needs consistent generation strategy in Laravel/PostgreSQL
- developers must avoid assuming sequential numeric IDs

---

## Implementation Notes

All main tables should use:

```text
id UUIDv7 PRIMARY KEY
```

Public references may still use human-friendly strings such as:

```text
PAY_20260624_XXXX
SET_20260624_XXXX
```

UUIDv7 is for internal primary keys. Public references are for support, merchant communication, and reconciliation.

---

## Rule

Use UUIDv7 for primary keys. Use explicit business references for human-facing identifiers.
