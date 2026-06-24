# 04 — Outbox Pattern

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how the platform persists and dispatches critical events reliably.

The Outbox Pattern prevents lost events when a database transaction succeeds but queue dispatch fails.

---

## 2. Problem

A common failure:

```text
1. Payment succeeds.
2. Database commit succeeds.
3. Queue dispatch fails.
4. Merchant never receives webhook.
```

This creates inconsistent operational behavior.

---

## 3. Decision

Persist critical events into an outbox table inside the same database transaction as the business state change.

A worker later reads the outbox and dispatches jobs/events.

---

## 4. Flow

```text
Business Transaction Starts
    ↓
Update Business Record
    ↓
Insert Outbox Event
    ↓
Commit Transaction
    ↓
Outbox Worker Reads Event
    ↓
Dispatch Webhook / Notification / Projection
    ↓
Mark Outbox Event Processed
```

---

## 5. Candidate Table: outbox_events

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

## 6. Critical Events

Use outbox for:

```text
PaymentSucceeded
PaymentFailed
RefundSucceeded
RefundFailed
SettlementCompleted
WebhookMarkedDead
LedgerPostingFailed
```

---

## 7. Worker Behavior

Outbox worker should:

1. select pending events
2. mark as processing
3. dispatch relevant handler/job
4. mark processed on success
5. increase attempt count on failure
6. schedule retry
7. mark dead after max attempts

---

## 8. Idempotency

Outbox handlers must be idempotent.

Examples:

- webhook delivery should not create duplicate business event
- notification can be deduplicated by event id
- ledger posting must prevent duplicate source posting

---

## 9. Transaction Boundary

The outbox event must be inserted in the same database transaction as the business state change.

Example:

```text
Payment status changed to SUCCEEDED
and
PaymentSucceeded event inserted
```

must commit together.

---

## 10. What Outbox Does Not Solve

Outbox does not replace:

- idempotency keys
- provider verification
- ledger posting validation
- state machines
- reconciliation

It only improves reliability of event dispatch.

---

## 11. Monitoring

Monitor:

- number of pending events
- number of failed events
- dead outbox events
- processing latency
- retry count

Dead critical events must alert operations/engineering.

---

## 12. Critical Rule

Any event that affects merchant notification, ledger follow-up, settlement, or reconciliation should be durable before dispatch.
