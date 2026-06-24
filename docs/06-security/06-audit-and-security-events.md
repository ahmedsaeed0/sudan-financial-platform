# 06 — Audit and Security Events

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines audit logs and security events.

Auditability is mandatory for financial, security, merchant, refund, settlement, and provider-related actions.

---

## 2. Audit vs Security Events

### Audit Log

Records sensitive business or operational actions.

Examples:

- refund approved
- merchant suspended
- fee profile changed
- settlement retried

### Security Event

Records security-related events.

Examples:

- login failed
- API authentication failed
- API key revoked
- rate limit exceeded

---

## 3. Audit Log Core Fields

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

---

## 4. Actor Types

```text
INTERNAL_USER
MERCHANT_USER
SYSTEM
JOB
```

---

## 5. Target Types

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
PROVIDER_CONFIGURATION
```

---

## 6. Required Audit Actions

Merchant:

```text
MERCHANT_APPROVED
MERCHANT_REJECTED
MERCHANT_SUSPENDED
MERCHANT_REACTIVATED
MERCHANT_PAYOUT_DESTINATION_CHANGED
```

API keys:

```text
API_KEY_CREATED
API_KEY_DISABLED
API_KEY_REVOKED
API_SECRET_ROTATED
API_SECRET_REVOKED
```

Financial operations:

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

Configuration:

```text
FEE_PROFILE_CHANGED
RISK_PROFILE_CHANGED
WEBHOOK_ENDPOINT_CREATED
WEBHOOK_ENDPOINT_DISABLED
```

---

## 7. Security Event Types

```text
LOGIN_SUCCESS
LOGIN_FAILED
PASSWORD_RESET_REQUESTED
API_AUTH_FAILED
API_RATE_LIMITED
API_KEY_USED
API_KEY_REVOKED
ADMIN_LOGIN
SUSPICIOUS_ACTIVITY_DETECTED
WEBHOOK_SIGNATURE_FAILED
```

---

## 8. Immutability

Audit logs should be append-only.

Do not update or delete audit records in normal application flows.

If correction is needed, add another audit entry.

---

## 9. Search Requirements

Audit logs should be searchable by:

```text
actor_type + actor_id
target_type + target_id
action
occurred_at
```

---

## 10. Sensitive Data Rules

Audit logs must not store secrets.

Avoid storing:

- API secrets
- webhook secrets
- authorization headers
- provider credentials

Use masked values or metadata instead.

---

## 11. Operational Alerts

Some security events should trigger alerts:

```text
many failed admin logins
API auth failures spike
rate limit abuse
provider auth failure
high-risk merchant action
ledger posting failure
settlement retry failure
```

---

## 12. Critical Rules

1. Sensitive actions must be audited.
2. Audit logs must not contain secrets.
3. Audit logs are append-only.
4. Security events should support alerting.
5. Financial manual actions require audit trail.
