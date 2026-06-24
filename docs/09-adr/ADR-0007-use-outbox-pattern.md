# ADR-0007 — Use Outbox Pattern

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

Many important actions create side effects:

- payment succeeded -> send webhook
- refund succeeded -> send webhook
- settlement completed -> notify merchant
- webhook failed -> schedule retry

A dangerous failure can happen:

```text
Database commit succeeds, but queue dispatch fails.
```

This can cause lost events.

---

## Decision

Use the Outbox Pattern for critical domain events.

Critical events should be persisted in the database first, then dispatched asynchronously by a worker.

---

## Alternatives Considered

### Dispatch jobs directly after database write

Pros:

- simple
- common Laravel pattern

Cons:

- queue dispatch can fail after DB commit
- event can be lost
- hard to recover reliably

### Synchronous side effects inside request

Pros:

- immediate
- simple mental model

Cons:

- slow response
- provider/webhook failures affect user flow
- poor reliability

### Outbox Pattern

Pros:

- durable event storage
- retryable dispatch
- recoverable
- safer for financial events

Cons:

- more infrastructure
- workers required
- requires cleanup/monitoring

---

## Consequences

Positive:

- prevents lost critical events
- improves webhook reliability
- improves operational recovery
- supports auditability

Negative:

- more tables and workers
- more monitoring required

---

## Implementation Notes

Potential table:

```text
outbox_events
```

Core fields:

```text
id
event_type
aggregate_type
aggregate_id
payload_json
status
attempts
next_attempt_at
processed_at
created_at
updated_at
```

Statuses:

```text
PENDING
PROCESSING
PROCESSED
FAILED
DEAD
```

---

## Critical Events

Use outbox for:

- PaymentSucceeded
- PaymentFailed
- RefundSucceeded
- RefundFailed
- SettlementCompleted
- WebhookMarkedDead

---

## Rule

Do not rely only on immediate queue dispatch for critical financial/domain events.
