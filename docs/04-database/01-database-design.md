# 01 — Database Design

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the database design direction for the Sudan Financial Platform MVP.

The goal is to design a database that supports:

- Merchant onboarding
- API keys and developer access
- Payments
- Payment links and hosted checkout
- Refunds
- Webhooks
- Ledger
- Settlement
- Reconciliation
- Risk
- Notifications
- Audit logs

This is a design document, not final migration code.

---

## 2. Core Database Principles

### 2.1 PostgreSQL First

The primary database is PostgreSQL.

Reasons:

- Strong ACID guarantees
- Reliable transactions
- Row-level locking
- Partial indexes
- JSONB support
- Long-term scaling path
- Good fit for financial systems

### 2.2 UUIDv7 Everywhere

Every primary key should use UUIDv7.

Reasons:

- Globally unique
- Time-sortable
- Better index locality than UUIDv4
- Safer for distributed systems

### 2.3 Money as Integer Minor Units

Never use float or double for money.

All money values must be stored as integers in minor units.

Examples:

```text
amount
fee_amount
net_amount
balance_amount
```

Currency must be stored separately.

### 2.4 Currency as CHAR(3)

Currency should be stored using a 3-character currency code.

MVP supports:

```text
SDG
```

The database should still allow future multi-currency support.

### 2.5 Ledger Is Financial Source of Truth

Payments are business records.

Ledger is the financial source of truth.

Do not calculate merchant balances using payment rows only.

Balances must come from ledger entries or ledger-derived projections.

### 2.6 No Hard Delete for Financial Records

Financial and operational records must not be hard deleted.

Use:

- status fields
- archived_at where needed
- audit logs
- reversal entries
- adjustment entries

### 2.7 State Machines for Financial Entities

The following must use controlled state transitions:

- Merchant
- Payment
- Refund
- Settlement
- Webhook Delivery
- Reconciliation Incident

Direct uncontrolled status changes are forbidden.

### 2.8 Idempotency by Default

Create Payment API must support idempotency.

The database must store idempotency records and prevent duplicate effects.

### 2.9 Audit Sensitive Actions

Audit must cover:

- Merchant approval
- Merchant suspension
- Fee changes
- API key creation
- API key rotation
- API key revocation
- Refund approval
- Settlement retry
- Risk profile changes
- Manual operational actions

---

## 3. Table Groups

| Group | Purpose |
|---|---|
| Identity | Users, roles, API keys, secrets |
| Merchant | Merchant profile, documents, payout destinations |
| Fee | Merchant fee profiles and fee snapshots |
| Risk | Risk limits, alerts, rule decisions |
| Payment | Payments, payment links, checkout sessions, idempotency |
| Provider | Provider attempts and normalized responses |
| Refund | Refund requests, approvals, execution status |
| Webhook | Endpoints, events, deliveries, attempts |
| Ledger | Accounts, journal entries, ledger entries, balances |
| Settlement | Settlement batches, items, attempts |
| Reconciliation | Reports, lines, incidents |
| Notification | In-app, email, and internal notifications |
| Audit | Sensitive action logs and security events |

---

## 4. Naming Conventions

### 4.1 Tables

Use plural snake_case names:

```text
merchants
payments
refunds
journal_entries
ledger_entries
```

### 4.2 Primary Keys

All tables use:

```text
id UUIDv7 PRIMARY KEY
```

### 4.3 Timestamps

Standard tables should use:

```text
created_at
updated_at
```

Financial and audit tables may also use:

```text
occurred_at
posted_at
processed_at
```

### 4.4 Status Fields

Use explicit status strings or application-level enums.

Database enum types can be considered later after the domain stabilizes.

---

## 5. Tenant Model

`merchants` is the primary tenant entity.

Most business records should have `merchant_id` directly or indirectly.

Merchant-scoped tables include:

- merchant_users
- merchant_documents
- merchant_payout_accounts
- api_keys
- fee_profiles
- risk_profiles
- payments
- payment_links
- refunds
- webhook_endpoints
- settlements

---

## 6. Critical Unique Constraints

| Table | Constraint | Reason |
|---|---|---|
| api_keys | public_key unique | identify API key safely |
| payments | reference unique | global support and reconciliation reference |
| payments | merchant_id + order_id unique | prevent duplicate merchant order payments |
| idempotency_keys | merchant_id + api_key_id + endpoint + idempotency_key unique | safe retries |
| settlements | reference unique | settlement tracking |
| journal_entries | reference unique | financial event tracking |
| ledger_accounts | code unique | stable accounting references |

---

## 7. Critical Indexing Strategy

Indexes should be based on access patterns.

Important lookup patterns:

- Search payment by reference
- Search payment by merchant and order_id
- Search payment by customer phone
- Search provider reference
- List payments by merchant and date
- List settlements by merchant and status
- Find pending webhook deliveries
- Find pending refunds
- Query ledger by merchant, date, and account
- Query audit by actor, target, and action

---

## 8. Data Sensitivity Classification

| Data Type | Sensitivity | Notes |
|---|---|---|
| Merchant profile | Medium | business data |
| KYC documents | High | restricted access |
| Payout account identifiers | High | mask and protect |
| API secrets | Critical | store hash only |
| Webhook secrets | High | store encrypted or secret reference |
| Payment customer snapshot | Medium/High | protect phone and email |
| Ledger entries | Critical | immutable financial truth |
| Audit logs | High | tamper-resistant behavior preferred |

---

## 9. Migration Philosophy

Do not generate all migrations blindly.

Recommended migration order:

1. Identity and users
2. Merchants
3. Merchant documents and payout destinations
4. API keys and secrets
5. Fee profiles
6. Risk profiles
7. Payments, links, checkout, idempotency
8. Provider attempts
9. Refunds
10. Webhooks
11. Ledger
12. Settlements
13. Reconciliation
14. Notifications
15. Audit and security logs

---

## 10. Review Rule

No migration should be created until:

- table purpose is clear
- ownership domain is clear
- relationships are clear
- indexes are justified
- sensitive fields are classified
- lifecycle/statuses are defined
- audit requirements are known
