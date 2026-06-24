# 02 — API Key Security

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how API keys are created, stored, rotated, revoked, and used securely.

API keys are the primary authentication mechanism for the Public Merchant API.

---

## 2. Key Types

Suggested key prefixes:

```text
pk_test_
sk_test_
pk_live_
sk_live_
```

### Public Key

May be used for public identification in future frontend-safe flows.

### Secret Key

Used by merchant backend to authenticate API requests.

Secret key must be protected by merchant.

---

## 3. Modes

```text
TEST
LIVE
```

Rules:

- TEST keys use MockProvider.
- LIVE keys use real provider integration.
- LIVE keys require approved merchant.

---

## 4. Statuses

```text
ACTIVE
DISABLED
REVOKED
```

Rules:

- ACTIVE can authenticate.
- DISABLED is temporarily blocked.
- REVOKED is final and should not be restored.

---

## 5. Storage Rule

Never store secret keys in plaintext.

Store only:

```text
secret_hash
```

Secret key is shown once during creation or rotation.

---

## 6. Secret Generation

Secrets must be:

- random
- high entropy
- not guessable
- long enough for API authentication
- generated server-side

Do not allow merchant-supplied API secrets.

---

## 7. Secret Rotation

Rotation flow:

1. Merchant developer requests rotation.
2. Platform generates new secret.
3. New secret becomes active.
4. Old secret remains active during grace period.
5. Old secret expires or is revoked.
6. Audit event is recorded.

---

## 8. Grace Period

Grace period prevents downtime during deployment.

During grace period:

- old secret works
- new secret works
- dashboard clearly shows rotation deadline

After grace period:

- old secret is revoked/expired

---

## 9. API Authentication Flow

1. Extract Bearer token.
2. Hash candidate secret.
3. Lookup active matching secret hash.
4. Resolve API key.
5. Check API key status.
6. Check merchant status.
7. Check mode.
8. Continue request if allowed.

---

## 10. LIVE Access Rule

LIVE API key can create live payments only if:

- API key is ACTIVE
- secret is ACTIVE
- merchant is APPROVED
- merchant is not SUSPENDED

---

## 11. Audit Events

Audit:

```text
API_KEY_CREATED
API_KEY_DISABLED
API_KEY_REVOKED
API_SECRET_ROTATED
API_SECRET_REVOKED
API_SECRET_EXPIRED
```

Audit should record:

- actor
- merchant
- api_key_id
- action
- timestamp
- IP address
- user agent

---

## 12. Logging Rules

Never log:

- secret key
- secret hash
- bearer token
- authorization header

Logs may include:

- api_key_id
- merchant_id
- public_key prefix
- request_id

---

## 13. Dashboard Rules

Dashboard should:

- show public key
- never show existing secret again
- show secret once on creation
- show last used timestamp
- allow disable/revoke
- allow rotation

---

## 14. Critical Rules

1. Secret keys are stored as hashes only.
2. Secret key is shown once.
3. LIVE keys require approved merchant.
4. Rotation must be audited.
5. Authorization header must never be logged.
