# 01 — System Architecture

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the high-level technical architecture for the Sudan Financial Platform MVP.

The system is designed as a financial infrastructure platform, not a simple payment form.

---

## 2. Architecture Style

The MVP uses:

```text
Laravel Modular Monolith
PostgreSQL
Redis
Queues
Provider Abstraction
Double Entry Ledger
```

Microservices are not recommended for MVP.

---

## 3. High-Level Architecture

```text
Client / Merchant Backend / Customer Browser
        ↓
Cloudflare
        ↓
Nginx
        ↓
Laravel Modular Monolith
        ↓
PostgreSQL + Redis
        ↓
Provider Layer
        ↓
MockProvider / EbsProvider / Future Providers
```

---

## 4. Edge Layer

### Cloudflare

Responsibilities:

- DNS
- HTTPS
- WAF
- DDoS protection
- basic rate limiting
- caching static assets

Cloudflare is not the source of application security.

Application-level authentication and authorization are still required.

---

## 5. Web Server Layer

### Nginx

Responsibilities:

- reverse proxy
- static file serving
- request forwarding to PHP runtime
- upload limits
- timeout configuration

---

## 6. Application Layer

### Laravel Modular Monolith

The application is one deployable unit divided into modules.

Core modules:

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

---

## 7. Runtime Strategy

Initial runtime:

```text
PHP-FPM
```

Future option:

```text
Laravel Octane or FrankenPHP
```

Do not adopt Octane before confirming correctness, memory behavior, worker lifecycle, and operational needs.

---

## 8. Database Layer

Primary database:

```text
PostgreSQL
```

Used for:

- merchants
- payments
- refunds
- settlements
- ledger
- reconciliation
- audit logs
- webhooks

PostgreSQL is the source of truth.

---

## 9. Redis Layer

Redis is used for:

- queues
- cache
- rate limiting
- short-lived locks

Redis is not used as financial truth.

Any financial invariant must be protected in PostgreSQL.

---

## 10. Queue Layer

Queues handle slow or retryable operations:

- webhook delivery
- notifications
- reconciliation processing
- settlement processing
- provider status verification
- report generation

Critical financial operations must be persisted first before side effects are queued.

---

## 11. Provider Layer

External payment providers are accessed through a provider abstraction.

```text
Payment Domain
    ↓
PaymentProvider Contract
    ↓
MockProvider / EbsProvider / Future Providers
```

Payment core must not depend directly on EBS implementation details.

---

## 12. Financial Core

Financial truth is handled by the Ledger Domain.

Core financial model:

```text
LedgerAccount
JournalEntry
LedgerEntry
BalanceProjection
```

Rule:

```text
Total Debit = Total Credit
```

---

## 13. API Surfaces

Main API surfaces:

```text
Public Merchant API
Merchant Dashboard API
Internal Admin API
Webhook Delivery Outbound API
```

Public APIs must be versioned:

```text
/api/v1
```

---

## 14. Environments

Required environments:

```text
Local
Staging / Sandbox
Production
```

No direct local-to-production deployment.

---

## 15. Observability

MVP observability should include:

- application error tracking
- queue monitoring
- failed job tracking
- webhook failure tracking
- provider failure tracking
- settlement failure tracking
- critical Telegram/internal alerts

---

## 16. Scaling Path

Stage 1:

```text
Single app server + PostgreSQL + Redis
```

Stage 2:

```text
Separate database and Redis
```

Stage 3:

```text
Separate worker servers
```

Stage 4:

```text
Multiple app servers behind load balancer
```

Stage 5:

```text
Selective service extraction if measured bottlenecks justify it
```

---

## 17. Critical Architecture Rules

1. PostgreSQL is source of truth.
2. Redis is not source of financial truth.
3. Ledger is financial source of truth.
4. Payments are business records.
5. Webhooks are asynchronous.
6. Provider logic is behind contract.
7. State machines control lifecycle changes.
8. No hard delete for financial records.
9. Critical events use durable persistence.
10. Do not start with microservices.
