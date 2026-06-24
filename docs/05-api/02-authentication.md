# 02 — Authentication

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines authentication models for the platform APIs and dashboards.

The platform has different actors:

- merchant backend systems
- merchant dashboard users
- internal admin users
- webhook receivers

---

## 2. Public Merchant API Authentication

Public Merchant API uses secret API keys.

Header:

```http
Authorization: Bearer <secret_key>
```

Example:

```http
Authorization: Bearer sk_test_xxx
```

---

## 3. API Key Types

Suggested key prefixes:

```text
pk_test_
sk_test_
pk_live_
sk_live_
```

### Public Key

May be used in frontend-safe contexts if needed later.

### Secret Key

Used by merchant backend.

Must be protected.

---

## 4. API Key Modes

```text
TEST
LIVE
```

Rules:

- TEST key routes to MockProvider.
- LIVE key routes to EbsProvider initially.
- LIVE key requires approved merchant.

---

## 5. API Key Statuses

```text
ACTIVE
DISABLED
REVOKED
```

Rules:

- ACTIVE keys can authenticate.
- DISABLED keys are temporarily blocked.
- REVOKED keys cannot be restored.

---

## 6. Secret Storage

Never store secret key in plaintext.

Store:

```text
secret_hash
```

Secret is shown once during creation or rotation.

---

## 7. Secret Rotation

Rotation should support grace period.

Flow:

1. Developer requests rotation.
2. New secret is generated.
3. New secret becomes active.
4. Old secret remains active during grace period.
5. Old secret expires or is revoked.
6. Audit log is recorded.

---

## 8. Merchant Dashboard Authentication

Merchant dashboard users authenticate with email/password or future SSO.

Merchant user roles:

```text
OWNER
ADMIN
DEVELOPER
FINANCE
SUPPORT
VIEWER
```

Authorization must check both:

- authenticated user
- merchant membership
- role permission

---

## 9. Internal Admin Authentication

Internal users authenticate separately from merchant users.

Internal roles:

```text
SUPER_ADMIN
OPERATIONS
FINANCE
RISK
SUPPORT
DEVELOPER
VIEWER
```

High-risk actions should be audited.

Future:

- 2FA
- maker-checker approvals
- IP restrictions

---

## 10. Webhook Authentication

Webhooks are authenticated by signature, not API key.

Headers:

```http
X-Signature
X-Timestamp
X-Event-Id
```

Algorithm:

```text
HMAC SHA256
```

Merchant verifies the signature using webhook secret.

---

## 11. Rate Limiting

Apply rate limits by:

- IP
- merchant
- API key
- endpoint

High-risk endpoints need stricter limits.

---

## 12. Failed Authentication

Failed authentication returns:

```text
401 UNAUTHORIZED
```

Disabled/revoked key returns:

```text
403 FORBIDDEN
```

Rate limit exceeded returns:

```text
429 RATE_LIMITED
```

---

## 13. Audit Requirements

Audit:

- API key created
- API key disabled
- API key revoked
- API secret rotated
- internal user login
- merchant role changed
- high-risk permission changed

---

## 14. Critical Rules

1. API secrets are stored as hashes only.
2. LIVE keys require approved merchant.
3. TEST and LIVE modes are separated.
4. Secret rotation is auditable.
5. Webhook authentication uses signatures.
