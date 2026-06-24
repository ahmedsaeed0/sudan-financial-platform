# ADR-0008 — Use State Machines

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

Financial entities move through strict lifecycles.

Examples:

- Payment
- Refund
- Settlement
- Merchant
- Webhook Delivery

If developers can randomly assign statuses, invalid financial states can occur.

Example problem:

```text
CREATED -> SUCCEEDED
```

without provider confirmation, ledger posting, or audit.

---

## Decision

Use state machines for financial and operational lifecycle entities.

Statuses must change through explicit transitions.

---

## Alternatives Considered

### Direct status assignment

Pros:

- simple
- fast to code

Cons:

- invalid states
- hidden side effects
- hard to audit
- difficult debugging

### State machine transitions

Pros:

- clear lifecycle
- prevents invalid transitions
- easier testing
- easier auditing
- safer financial behavior

Cons:

- more initial design
- developers must use transition methods

---

## Entities Requiring State Machines

```text
Merchant
Payment
Refund
Settlement
WebhookDelivery
ReconciliationIncident
```

---

## Payment States

```text
CREATED
PENDING
PROCESSING
SUCCEEDED
FAILED
EXPIRED
CANCELLED
REFUNDED
```

---

## Refund States

```text
REQUESTED
PENDING_APPROVAL
APPROVED
REJECTED
PROCESSING
SUCCEEDED
FAILED
```

---

## Settlement States

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

---

## Consequences

Positive:

- safer lifecycle control
- fewer financial bugs
- clear tests
- better auditability

Negative:

- more code structure
- requires transition policies

---

## Implementation Notes

Use explicit methods such as:

```text
markProcessing()
markSucceeded()
markFailed()
expire()
cancel()
refund()
```

Do not allow arbitrary status mutation from controllers.

---

## Rule

No direct uncontrolled status assignment for financial lifecycle entities.
