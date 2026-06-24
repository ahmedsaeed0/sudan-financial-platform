# 03 — Database Relationships

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the main database relationships for the MVP.

The goal is to make ownership and data boundaries explicit before writing migrations.

---

## 2. Tenant Root

`merchants` is the primary tenant root.

Most business data must be scoped to a merchant directly or indirectly.

---

## 3. Merchant Relationships

```text
merchants
├── merchant_users
├── merchant_documents
├── merchant_payout_accounts
├── api_keys
├── fee_profiles
├── risk_profiles
├── payments
├── refunds
├── webhook_endpoints
└── settlements
```

### Rules

- Merchant owns its payment records.
- Merchant owns its API keys.
- Merchant owns its webhook endpoints.
- Merchant owns its settlement configuration and payout destinations.
- Merchant status controls LIVE payment ability.

---

## 4. Identity Relationships

```text
merchants
└── merchant_users

api_keys
└── api_key_secrets
```

### Rules

- Merchant users belong to one merchant.
- Internal users are separate from merchant users.
- API secrets belong to API keys.
- API secrets are hashed only.

---

## 5. Payment Relationships

```text
merchants
└── payments

api_keys
└── payments

payments
├── checkout_sessions
├── refunds
└── journal_entries through source reference
```

### Rules

- A payment belongs to one merchant.
- A payment is created using one API key or dashboard action.
- A payment may have one refund in MVP.
- Payment financial impact is represented through ledger journal entries.

---

## 6. Payment Link Relationships

```text
merchants
└── payment_links

payment_links
└── payments nullable
```

### Rules

- Payment links belong to merchants.
- A payment link may create or reference a payment.
- Payment link status is separate from payment status.

---

## 7. Refund Relationships

```text
payments
└── refunds

merchants
└── refunds

refunds
└── journal_entries through source reference
```

### Rules

- Refund belongs to payment.
- Refund also stores merchant_id for tenant-scoped queries.
- Refund execution must post ledger entries.

---

## 8. Webhook Relationships

```text
merchants
└── webhook_endpoints
    └── webhook_deliveries
```

### Rules

- Webhook endpoints belong to merchants.
- Webhook deliveries belong to webhook endpoints.
- Deliveries are retried independently.

---

## 9. Ledger Relationships

```text
journal_entries
└── ledger_entries
    └── ledger_accounts
```

### Source Reference

Journal entries link to business records using:

```text
source_type
source_id
posting_type
```

Examples:

```text
PAYMENT + payment_id + PAYMENT_SUCCESS
REFUND + refund_id + REFUND_SUCCESS
SETTLEMENT + settlement_id + SETTLEMENT_COMPLETED
```

### Rules

- Ledger entries belong to one journal entry.
- Ledger entries belong to one ledger account.
- Journal entry must balance.
- Posted ledger records are immutable.

---

## 10. Settlement Relationships

```text
merchants
└── settlements

merchant_payout_accounts
└── settlements

settlements
└── journal_entries through source reference
```

### Rules

- Settlement belongs to one merchant.
- Settlement targets one payout destination.
- Settlement completion posts ledger entries.

---

## 11. Reconciliation Relationships

```text
reconciliation_reports
└── reconciliation_incidents
```

### Rules

- Report contains reconciliation run metadata.
- Incidents represent mismatches.
- Incidents should reference internal and external references.

---

## 12. Notification Relationships

Notifications may reference users or internal actors using polymorphic-style fields:

```text
recipient_type
recipient_id
```

Examples:

```text
MERCHANT_USER + merchant_user_id
INTERNAL_USER + internal_user_id
```

---

## 13. Audit Relationships

Audit logs use generic actor and target references:

```text
actor_type
actor_id
target_type
target_id
```

Examples:

```text
actor_type = INTERNAL_USER
actor_id = internal_users.id
target_type = REFUND
target_id = refunds.id
```

---

## 14. Relationship Rules

1. Merchant-owned records must be tenant-scoped.
2. Ledger must link back to source business record.
3. Audit must link actor and target.
4. Webhook delivery must be traceable to endpoint and event.
5. Refund and settlement must be traceable to ledger postings.
6. Avoid random cross-module table access in application code.
