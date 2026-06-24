# 03 — Event Flow

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how domain events and side effects flow through the system.

Events help decouple modules, but they must not hide critical financial logic.

---

## 2. Event Philosophy

Use events for:

- side effects
- notifications
- webhooks
- reporting projections
- async processing
- operational alerts

Do not use events to hide mandatory financial logic.

Financial state changes must be explicit and transactional.

---

## 3. Event Naming

Events should be past tense.

Examples:

```text
PaymentCreated
PaymentSucceeded
RefundApproved
SettlementCompleted
WebhookDeliveryFailed
LedgerEntryPosted
```

---

## 4. Critical Payment Event Flow

```text
PaymentSucceeded
    ↓
LedgerPostingRequested
    ↓
JournalEntryPosted
    ↓
WebhookEventCreated
    ↓
NotificationQueued
    ↓
ReportingProjectionUpdated
```

Important:

Ledger posting should not be treated as optional side effect.

If ledger posting fails, this is a critical incident.

---

## 5. Refund Event Flow

```text
RefundApproved
    ↓
RefundProcessing
    ↓
ProviderRefundSucceeded
    ↓
RefundSucceeded
    ↓
RefundLedgerPostingRequested
    ↓
RefundWebhookQueued
    ↓
RefundNotificationQueued
```

---

## 6. Settlement Event Flow

```text
SettlementCreated
    ↓
SettlementProcessing
    ↓
SettlementCompleted
    ↓
SettlementLedgerPostingRequested
    ↓
SettlementNotificationQueued
    ↓
SettlementReportGenerated
```

---

## 7. Webhook Event Flow

```text
Domain Event
    ↓
Webhook Event Stored
    ↓
Webhook Delivery Queued
    ↓
Webhook Attempted
    ↓
Delivered / Retry Scheduled / Dead
```

Webhook delivery failure must not rollback the original business event.

---

## 8. Risk Event Flow

```text
RiskCheckStarted
    ↓
RiskCheckPassed / RiskCheckFailed
    ↓
RiskAlertCreated if needed
```

Risk checks should happen before provider calls for payment creation.

---

## 9. Event Types

### Domain Events

Represent important business facts.

Examples:

```text
PaymentSucceeded
RefundApproved
SettlementCompleted
MerchantSuspended
```

### Integration Events

Events intended for external systems.

Examples:

```text
payment.succeeded
refund.succeeded
settlement.completed
```

### Operational Events

Events for internal monitoring.

Examples:

```text
ProviderTimedOut
WebhookMarkedDead
SettlementFailed
LedgerPostingFailed
```

---

## 10. Synchronous vs Asynchronous

### Synchronous

Use synchronous flow for:

- request validation
- idempotency check
- merchant status check
- risk check before payment creation
- database write for payment creation

### Asynchronous

Use asynchronous flow for:

- webhook delivery
- email notifications
- report generation
- reconciliation processing
- provider status verification

---

## 11. Event Storage

Critical events should be persisted using Outbox Pattern.

This protects against event loss when:

- database commit succeeds
- queue dispatch fails
- worker crashes

---

## 12. Failure Handling

If event processing fails:

- record failure
- retry if safe
- mark as dead if retries exhausted
- alert operations for critical events

Critical events:

```text
LedgerPostingFailed
WebhookMarkedDead
SettlementFailed
ProviderTimedOut
```

---

## 13. Idempotent Event Processing

Event consumers should be idempotent.

Example:

A webhook worker retry should not create duplicate business records.

A ledger posting handler must not post the same source event twice.

---

## 14. Critical Rules

1. Events represent facts that already happened.
2. Critical events must be persisted.
3. Event consumers must be idempotent.
4. Webhook failure does not rollback payment success.
5. Ledger failure is critical and must be visible.
6. Do not hide financial consistency behind uncontrolled events.
