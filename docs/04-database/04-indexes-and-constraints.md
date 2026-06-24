# 04 — Indexes and Constraints

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the first indexing and constraint strategy for the MVP database.

Indexes should support real access patterns, not be added randomly.

Constraints should protect business invariants at the database level where possible.

---

## 2. Constraint Philosophy

Use database constraints to protect rules that must never be violated.

Examples:

- payment reference uniqueness
- merchant order uniqueness
- API public key uniqueness
- ledger account code uniqueness
- journal entry reference uniqueness
- idempotency key uniqueness

Application validation is not enough for financial systems.

---

## 3. Primary Keys

Every table uses:

```text
id UUIDv7 PRIMARY KEY
```

Reason:

- unique across systems
- sortable by time
- better locality than UUIDv4

---

## 4. Foreign Keys

Foreign keys should be used for core relational integrity.

Examples:

```text
merchant_users.merchant_id -> merchants.id
api_keys.merchant_id -> merchants.id
api_key_secrets.api_key_id -> api_keys.id
payments.merchant_id -> merchants.id
payments.api_key_id -> api_keys.id
refunds.payment_id -> payments.id
refunds.merchant_id -> merchants.id
webhook_endpoints.merchant_id -> merchants.id
webhook_deliveries.webhook_endpoint_id -> webhook_endpoints.id
journal_entries -> source reference by type/id, no strict FK
ledger_entries.journal_entry_id -> journal_entries.id
ledger_entries.ledger_account_id -> ledger_accounts.id
settlements.merchant_id -> merchants.id
settlements.payout_account_id -> merchant_payout_accounts.id
```

Source references like `source_type/source_id` are intentionally generic and may not use strict FK.

---

## 5. Unique Constraints

### api_keys

```text
unique(public_key)
```

### payments

```text
unique(reference)
unique(merchant_id, order_id)
```

### idempotency_keys

```text
unique(merchant_id, api_key_id, endpoint, idempotency_key)
```

### ledger_accounts

```text
unique(code)
```

### journal_entries

```text
unique(reference)
unique(source_type, source_id, posting_type)
```

### settlements

```text
unique(reference)
```

### balance_projections

```text
unique(merchant_id, currency)
```

---

## 6. Payment Indexes

Important queries:

- find payment by reference
- find payment by merchant order ID
- list merchant payments by date
- filter merchant payments by status
- search by provider reference
- support by customer phone

Recommended indexes:

```text
payments(reference)
payments(merchant_id, order_id)
payments(merchant_id, created_at)
payments(merchant_id, status)
payments(provider_reference)
payments(customer_phone)
payments(status, expires_at)
```

The `status, expires_at` index supports payment expiry jobs.

---

## 7. Webhook Indexes

Important queries:

- find pending deliveries
- find deliveries ready for retry
- list endpoint deliveries

Recommended indexes:

```text
webhook_deliveries(status)
webhook_deliveries(next_retry_at)
webhook_deliveries(status, next_retry_at)
webhook_deliveries(webhook_endpoint_id, created_at)
```

---

## 8. Ledger Indexes

Important queries:

- get journal entries by source
- query ledger entries by account/date
- query merchant ledger entries by date
- build merchant balances

Recommended indexes:

```text
journal_entries(reference)
journal_entries(source_type, source_id)
journal_entries(source_type, source_id, posting_type)
journal_entries(posted_at)
ledger_entries(journal_entry_id)
ledger_entries(ledger_account_id, created_at)
ledger_entries(merchant_id, created_at)
ledger_entries(currency)
```

---

## 9. Settlement Indexes

Important queries:

- list settlements by merchant
- list pending settlements
- search by reference

Recommended indexes:

```text
settlements(reference)
settlements(merchant_id, created_at)
settlements(status)
settlements(status, processed_at)
```

---

## 10. Refund Indexes

Important queries:

- list pending refunds
- list merchant refunds
- find refund by payment

Recommended indexes:

```text
refunds(payment_id)
refunds(merchant_id, created_at)
refunds(status)
refunds(status, created_at)
```

---

## 11. Audit Indexes

Important queries:

- audit by actor
- audit by target
- audit by action
- audit by date

Recommended indexes:

```text
audit_logs(actor_type, actor_id)
audit_logs(target_type, target_id)
audit_logs(action)
audit_logs(occurred_at)
```

---

## 12. Partial Index Candidates

PostgreSQL partial indexes may be useful later.

Examples:

```text
webhook_deliveries where status in ('PENDING', 'RETRYING')
payments where status in ('CREATED', 'PENDING')
settlements where status in ('PENDING', 'FAILED')
refunds where status = 'PENDING_APPROVAL'
```

Do not overuse partial indexes before real query patterns are measured.

---

## 13. Constraint vs Application Rule

Some rules belong in database:

- uniqueness
- foreign key ownership
- non-null required fields
- positive money amounts

Some rules belong in application/domain layer:

- state transition validity
- fee calculation
- ledger balancing before insert
- risk decisions
- provider routing

Some rules should exist in both:

- no negative amount
- no duplicate payment order
- idempotency uniqueness

---

## 14. Critical Rule

Indexes are not decorations.

Every index must support a query or a constraint.

Every financial uniqueness rule should be enforced by the database where possible.
