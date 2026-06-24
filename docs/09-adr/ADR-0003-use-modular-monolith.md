# ADR-0003 — Use Modular Monolith

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

The platform will contain multiple domains:

- Identity
- Merchant
- Payment
- Provider
- Ledger
- Refund
- Settlement
- Reconciliation
- Webhook
- Risk
- Notification
- Audit
- Reporting

A common early mistake is starting with microservices before the product and domain boundaries are stable.

This creates unnecessary operational complexity.

---

## Decision

Start with a Laravel Modular Monolith.

The application is one deployable system, but internally divided into strong modules and domain boundaries.

---

## Alternatives Considered

### Traditional Laravel MVC Monolith

Pros:

- fastest to start
- familiar to most Laravel developers

Cons:

- domain logic tends to leak into controllers and models
- harder to scale team and codebase
- can become a large unstructured codebase

### Microservices From Day One

Pros:

- independent scaling
- independent deployments
- strong isolation if done well

Cons:

- too much operational overhead early
- distributed transactions are hard
- debugging is harder
- testing is harder
- not justified before real scale

### Modular Monolith

Pros:

- faster than microservices
- cleaner than classic MVC monolith
- strong consistency
- easier debugging
- easier refactoring before boundaries stabilize

Cons:

- requires discipline
- module boundaries can be violated if not enforced
- future extraction needs planning

---

## Consequences

Positive:

- faster delivery
- simpler deployment
- easier local development
- reliable transactions
- clearer domain boundaries

Negative:

- requires architecture discipline
- needs code review to prevent module leaks
- may require service extraction later

---

## Implementation Notes

Suggested modules:

```text
Identity
Merchant
Payment
Provider
Fee
Ledger
Refund
Settlement
Reconciliation
Webhook
Risk
Notification
Audit
Reporting
Infrastructure
```

Each module should contain:

```text
Application
Domain
Infrastructure
Presentation
Tests
```

---

## Future Extraction Candidates

Only after measured bottlenecks:

- Webhook delivery service
- Reporting service
- Risk service
- Settlement service
- Provider gateway service

---

## Rule

Do not start with microservices. Extract services only after metrics prove the need.
