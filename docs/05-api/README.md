# API Documentation

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

This directory defines the API design for the Sudan Financial Platform.

The public API must be stable, versioned, secure, idempotent where money can be affected, and developer-friendly.

---

## Documents

| File | Purpose |
|---|---|
| [01-api-design.md](./01-api-design.md) | General API design principles |
| [02-authentication.md](./02-authentication.md) | API key and dashboard authentication |
| [03-create-payment-api.md](./03-create-payment-api.md) | Create Payment API contract |
| [04-payment-status-api.md](./04-payment-status-api.md) | Payment retrieval and status APIs |
| [05-refund-api.md](./05-refund-api.md) | Refund request API |
| [06-webhook-payloads.md](./06-webhook-payloads.md) | Webhook event payloads and signature model |
| [07-error-codes.md](./07-error-codes.md) | Standard API error codes |
| [08-idempotency-behavior.md](./08-idempotency-behavior.md) | Idempotency API behavior |
| [09-api-versioning.md](./09-api-versioning.md) | Versioning and compatibility rules |

---

## Core Rules

1. Public APIs must be versioned.
2. Create Payment requires Idempotency-Key.
3. API secrets are never stored in plaintext.
4. Webhooks are signed.
5. Redirect is UX only; webhook is server truth.
6. Errors use a consistent response shape.
7. Money is represented as integer minor units.
8. Currency is always explicit.

---

## Base URL

Initial API base path:

```text
/api/v1
```
