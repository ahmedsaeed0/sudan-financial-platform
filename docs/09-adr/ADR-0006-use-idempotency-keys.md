# ADR-0006 — Use Idempotency Keys

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

Payment creation can be retried because of:

- client timeout
- network failure
- mobile app retry
- browser double click
- merchant backend retry
- proxy retry

Without idempotency, the same merchant order may create multiple payments.

---

## Decision

Require Idempotency-Key for Create Payment API.

The database must store idempotency records and enforce uniqueness.

---

## Alternatives Considered

### No idempotency

Pros:

- simpler API

Cons:

- duplicate payments
- unsafe retries
- poor developer experience

### Redis-only idempotency

Pros:

- fast
- simple short-term lock

Cons:

- not durable enough as source of truth
- can lose state
- not enough for financial invariants

### PostgreSQL-backed idempotency

Pros:

- durable
- transactional
- enforceable with unique constraint
- recoverable

Cons:

- more schema and logic

---

## Consequences

Positive:

- safe retries
- duplicate prevention
- better merchant integration experience
- protects against double-click and timeout retry issues

Negative:

- client must provide key
- requires request hashing
- requires conflict behavior

---

## Implementation Notes

Table:

```text
idempotency_keys
```

Unique scope:

```text
merchant_id + api_key_id + endpoint + idempotency_key
```

Store:

```text
request_hash
response_json
status
resource_type
resource_id
expires_at
```

Conflict rule:

```text
same key + different request_hash = IDEMPOTENCY_CONFLICT
```

---

## Rule

Idempotency is required for Create Payment API.

Idempotency does not replace concurrency control, ledger posting uniqueness, or database constraints.
