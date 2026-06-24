# 05 — Queue Strategy

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how queues are used in the platform.

Queues are used for slow, retryable, and side-effect operations.

Queues must not become the source of financial truth.

---

## 2. Queue Backend

Initial queue backend:

```text
Redis
```

Use Laravel queue workers and Laravel Horizon if available.

---

## 3. Queue Categories

Recommended queues:

```text
default
webhooks
notifications
settlement
reconciliation
provider-status
reports
outbox
```

---

## 4. Queue Responsibilities

### default

General background jobs that are not high-risk.

### webhooks

Webhook delivery and retries.

### notifications

Email, in-app notifications, internal alerts.

### settlement

Settlement processing and retry.

### reconciliation

Report parsing, matching, incident generation.

### provider-status

Provider status verification after timeout or pending state.

### reports

CSV exports and heavy report generation.

### outbox

Outbox event processing.

---

## 5. What Should Be Queued

Queue:

- webhook delivery
- notification sending
- reconciliation processing
- report generation
- settlement processing
- provider status verification
- dead webhook alerts

Do not queue without persistence:

- critical financial state changes
- ledger posting without source uniqueness
- payment creation business record

---

## 6. Payment Request Path

Payment creation request should remain relatively fast.

Synchronous:

1. validate request
2. validate API key
3. check merchant status
4. check idempotency
5. run risk checks
6. create payment
7. return response

Asynchronous:

- webhooks
- notifications
- reporting projections
- provider verification after timeout

---

## 7. Retry Rules

Jobs must define retry behavior intentionally.

Examples:

Webhook delivery:

```text
immediate
1 minute
5 minutes
15 minutes
1 hour
6 hours
then DEAD
```

Settlement retry:

```text
controlled retry only
must check duplicate payout risk
```

Provider status verification:

```text
retry until final provider status or reconciliation required
```

---

## 8. Idempotent Jobs

Queue jobs must be idempotent.

Examples:

- sending same webhook should not create duplicate domain event
- ledger posting job must not post same source twice
- settlement retry must not duplicate payout

---

## 9. Failed Jobs

Failed jobs must be visible.

Monitor:

- failed webhook jobs
- failed settlement jobs
- failed reconciliation jobs
- failed provider status jobs
- failed outbox jobs

Critical failures should alert operations/engineering.

---

## 10. Worker Scaling

Start simple:

```text
one or few workers on same app server
```

Then scale:

```text
separate worker server
separate workers per queue
priority workers
```

High-risk queues such as settlement should have carefully controlled concurrency.

---

## 11. Queue Priority

Suggested priority:

1. provider-status
2. settlement
3. webhooks
4. outbox
5. notifications
6. reports
7. reconciliation heavy jobs

Actual priority depends on production behavior.

---

## 12. Concurrency Control

Some jobs require locks:

- settlement processing per merchant
- ledger posting per source
- webhook delivery per delivery id
- reconciliation report processing

Use PostgreSQL constraints for final invariants.

Redis locks can help but should not be the only protection.

---

## 13. Critical Rules

1. Queues are for async execution, not financial truth.
2. Jobs must be idempotent.
3. Critical events should be persisted before dispatch.
4. Settlement jobs require careful duplicate payout protection.
5. Failed jobs must be observable.
