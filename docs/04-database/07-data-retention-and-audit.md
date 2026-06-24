# 07 — Data Retention and Audit

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines data retention, deletion, audit, and traceability rules.

Financial systems must preserve history.

Deleting or editing the wrong record can destroy reconciliation, settlement, support, and compliance confidence.

---

## 2. Core Rule

Financial records must not be hard deleted.

Use:

- status changes
- archived_at
- disabled_at
- reversal entries
- adjustment entries
- audit logs

---

## 3. Data Categories

| Category | Examples | Hard Delete? | Notes |
|---|---|---:|---|
| Financial records | payments, refunds, settlements, ledger entries | No | preserve history |
| Security records | API keys, secrets, security events | No, revoke instead | secrets stored as hash only |
| Operational records | webhook deliveries, provider attempts | Usually no | useful for support/debugging |
| Merchant profile | merchant data, settings | Usually no | archive/suspend instead |
| Documents | KYC files | Controlled retention | depends on policy/regulation |
| Notifications | in-app/email logs | can archive | not financial truth |

---

## 4. Tables That Must Not Be Hard Deleted

```text
payments
refunds
settlements
ledger_accounts
journal_entries
ledger_entries
reconciliation_reports
reconciliation_incidents
audit_logs
api_keys
api_key_secrets
merchant_payout_accounts
```

---

## 5. Tables That May Support Archival

```text
notifications
webhook_deliveries
provider_attempts
email_deliveries
```

Even for these, production deletion should be controlled by policy and retention period.

---

## 6. Audit Log Design

### Purpose

Track sensitive actions.

### Core Fields

```text
id
actor_type
actor_id
action
target_type
target_id
old_values_json
new_values_json
ip_address
user_agent
occurred_at
created_at
```

### Actor Types

```text
INTERNAL_USER
MERCHANT_USER
SYSTEM
JOB
```

### Target Types

```text
MERCHANT
API_KEY
API_SECRET
PAYMENT
REFUND
SETTLEMENT
FEE_PROFILE
RISK_PROFILE
WEBHOOK_ENDPOINT
RECONCILIATION_INCIDENT
```

---

## 7. Actions That Must Be Audited

Merchant actions:

```text
MERCHANT_APPROVED
MERCHANT_REJECTED
MERCHANT_SUSPENDED
MERCHANT_REACTIVATED
MERCHANT_BANK_ACCOUNT_UPDATED
```

API key actions:

```text
API_KEY_CREATED
API_KEY_DISABLED
API_KEY_REVOKED
API_SECRET_ROTATED
API_SECRET_REVOKED
```

Financial actions:

```text
REFUND_REQUESTED
REFUND_APPROVED
REFUND_REJECTED
REFUND_RETRIED
SETTLEMENT_CREATED
SETTLEMENT_RETRIED
SETTLEMENT_CANCELLED
MANUAL_ADJUSTMENT_CREATED
```

Configuration actions:

```text
FEE_PROFILE_CHANGED
RISK_PROFILE_CHANGED
WEBHOOK_ENDPOINT_CREATED
WEBHOOK_ENDPOINT_DISABLED
```

---

## 8. Audit Immutability

Audit logs should be append-only.

Avoid updating existing audit records except for exceptional operational metadata.

Do not delete audit logs in normal application flows.

---

## 9. Security Events

Security events should track suspicious or sensitive security-related activity.

Examples:

```text
LOGIN_FAILED
LOGIN_SUCCESS
PASSWORD_RESET_REQUESTED
API_AUTH_FAILED
API_RATE_LIMITED
API_KEY_USED
ADMIN_LOGIN
SUSPICIOUS_ACTIVITY_DETECTED
```

---

## 10. Document Retention

Merchant documents should be stored in object storage.

Database stores metadata and file references.

Rules:

- restrict access
- log access if possible
- avoid public URLs
- use temporary signed URLs where possible
- define retention policy before production

---

## 11. Ledger Retention

Ledger records are permanent financial records.

Rules:

- no hard delete
- no direct amount edits
- corrections through adjustment or reversal
- every posting references source entity

---

## 12. Soft Delete Policy

Soft delete can be used for non-financial convenience records, but avoid using it as a replacement for proper status modeling.

For financial objects, prefer explicit statuses:

```text
CANCELLED
FAILED
REVOKED
DISABLED
ARCHIVED
```

---

## 13. Retention Questions Before Production

Need decisions on:

1. How long to retain webhook delivery logs?
2. How long to retain provider attempt logs?
3. How long to retain KYC documents?
4. What data can be anonymized?
5. Who can access audit logs?
6. Do audit logs need export for compliance?

---

## 14. Critical Rule

If an action changes money, access, configuration, settlement, refund, merchant status, or provider behavior, it must be auditable.
