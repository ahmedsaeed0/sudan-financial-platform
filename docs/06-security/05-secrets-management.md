# 05 — Secrets Management

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how secrets are stored, accessed, rotated, and protected.

Secrets include API keys, provider credentials, webhook secrets, encryption keys, and operational credentials.

---

## 2. Secret Types

Secrets include:

```text
API secret keys
Webhook secrets
Provider API credentials
Database passwords
Redis passwords
Object storage credentials
Application encryption key
Mail provider credentials
Internal alert tokens
```

---

## 3. Core Rules

1. Never commit secrets to Git.
2. Never log secrets.
3. Never store API secrets in plaintext.
4. Restrict provider credentials.
5. Rotate secrets when exposed or no longer trusted.
6. Separate local, staging, and production secrets.

---

## 4. Environment Separation

Use separate secrets for:

```text
local
staging
production
```

Never reuse production credentials in local development.

---

## 5. Storage Strategy

MVP acceptable:

```text
.env on secured server
Laravel encrypted configuration where appropriate
hashed API secrets in database
encrypted webhook secrets or secret references
```

Future recommended:

```text
Vault
AWS Secrets Manager
Doppler
Infisical
1Password Secrets Automation
```

---

## 6. API Secrets

API secret rules:

- generated server-side
- shown once
- stored as hash only
- rotation supported
- revocation supported
- never logged

---

## 7. Webhook Secrets

Webhook secret rules:

- generated server-side
- used for HMAC signing
- protected at rest
- rotation supported in future
- never included in payload

---

## 8. Provider Credentials

Provider credentials must be isolated.

Rules:

- do not hardcode provider credentials
- do not store provider credentials in source code
- restrict access to production credentials
- rotate if developer leaves or exposure suspected
- log provider request references, not credentials

---

## 9. Logging Redaction

Logs must redact:

```text
Authorization header
API secret
Webhook secret
Provider credential
Database password
Object storage secret
Full sensitive payout details
```

Allowed in logs:

```text
request_id
merchant_id
payment_reference
api_key_id
provider_name
provider_reference
```

---

## 10. Secret Rotation Events

Audit:

```text
API_SECRET_ROTATED
WEBHOOK_SECRET_ROTATED
PROVIDER_CREDENTIAL_ROTATED
APP_KEY_ROTATION_PLANNED
```

---

## 11. Access Control

Only authorized internal roles should access production secret management.

Suggested roles:

```text
SUPER_ADMIN
ENGINEERING_LEAD
SECURITY_ADMIN future
```

---

## 12. Incident Response

If a secret is exposed:

1. revoke exposed secret
2. generate replacement
3. deploy/update configuration
4. audit incident
5. notify affected merchants if needed
6. review logs for abuse

---

## 13. Critical Rules

1. Secrets are not source code.
2. Secrets are not logs.
3. API secrets are hashed only.
4. Provider credentials are isolated.
5. Production secrets are restricted.
