# 02 — Modular Monolith Structure

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the Laravel Modular Monolith structure for the platform.

The goal is to avoid a random MVC monolith while also avoiding premature microservices.

---

## 2. Core Decision

Use one Laravel application as the deployable unit.

Internally, divide the application into strong domain modules.

---

## 3. Why Modular Monolith

Reasons:

- faster delivery than microservices
- simpler deployment
- easier local development
- stronger transactions
- easier debugging
- better domain separation than classic MVC
- easier future service extraction

---

## 4. Suggested Repository Structure

```text
backend/
frontend/
docs/
openapi/
postman/
scripts/
infrastructure/
```

---

## 5. Laravel Module Structure

Suggested:

```text
app/
modules/
config/
database/
routes/
tests/
```

Modules:

```text
modules/
├── Identity/
├── Merchant/
├── Payment/
├── Provider/
├── Fee/
├── Ledger/
├── Refund/
├── Settlement/
├── Reconciliation/
├── Webhook/
├── Risk/
├── Notification/
├── Audit/
├── Reporting/
└── Infrastructure/
```

---

## 6. Standard Module Layout

Each module should follow:

```text
ModuleName/
├── Application/
├── Domain/
├── Infrastructure/
├── Presentation/
└── Tests/
```

---

## 7. Application Layer

Coordinates use cases.

Examples:

```text
CreatePaymentHandler
ApproveRefundHandler
GenerateSettlementBatchHandler
PostPaymentLedgerHandler
SendWebhookHandler
```

Application layer may call:

- domain services
- repositories
- other module public contracts
- queues
- transactions

Application layer should not contain low-level HTTP or provider-specific implementation details.

---

## 8. Domain Layer

Contains business rules.

Examples:

```text
PaymentStateMachine
Money
FeeBreakdown
LedgerPosting
RefundPolicy
SettlementEligibility
RiskDecision
```

Domain layer should avoid Laravel-specific dependencies where practical.

---

## 9. Infrastructure Layer

Contains technical details:

- Eloquent models
- repositories
- provider clients
- jobs
- cache
- storage
- external HTTP clients

Examples:

```text
EloquentPaymentRepository
EbsProviderClient
SendWebhookJob
PostgresLedgerRepository
```

---

## 10. Presentation Layer

Contains HTTP/API interface:

- controllers
- form requests
- API resources
- routes

Examples:

```text
PaymentController
CreatePaymentRequest
PaymentResource
AdminRefundController
```

Controllers must not contain financial business logic.

---

## 11. Cross-Module Communication

Allowed:

- application services
- public contracts/interfaces
- domain events
- read models where explicitly allowed

Avoid:

- random Eloquent access across modules
- controllers calling another module's internals
- direct table access from unrelated modules

---

## 12. Example: Payment Success Flow

```text
Payment Module
  -> marks payment succeeded
  -> requests Ledger Module posting
  -> creates domain/outbox event
  -> Webhook Module delivers event
  -> Notification Module notifies merchant
```

---

## 13. Module Boundaries

### Payment Module

Owns payment lifecycle.

Does not own ledger internals.

### Ledger Module

Owns financial truth.

Does not own checkout UX.

### Provider Module

Owns external provider integration.

Does not own payment business lifecycle.

### Settlement Module

Owns settlement workflow.

Does not compute balance from payments table.

---

## 14. Routes

Suggested route files:

```text
routes/api_v1.php
routes/merchant.php
routes/admin.php
routes/internal.php
```

---

## 15. Tests

Suggested test groups:

```text
tests/Unit
tests/Feature
tests/Domain
tests/Integration
tests/Architecture
```

Architecture tests should verify module boundaries where possible.

---

## 16. Future Extraction

Potential future services:

- Webhook Delivery Service
- Reporting Service
- Risk Service
- Settlement Service
- Provider Gateway Service

Only extract after measured bottlenecks.

---

## 17. Critical Rules

1. Modular Monolith is not random folders.
2. Controllers do not contain business logic.
3. Financial logic belongs in domain/application layers.
4. Cross-module access must be intentional.
5. Extract services only after metrics justify it.
