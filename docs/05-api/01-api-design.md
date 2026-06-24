# 01 — API Design

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the general API design principles for the Sudan Financial Platform.

The API should be secure, predictable, stable, versioned, and friendly for merchant developers.

---

## 2. API Groups

Initial API groups:

```text
Public Merchant API
Merchant Dashboard API
Internal Admin API
Webhook Delivery API
```

---

## 3. Public API Base Path

```text
/api/v1
```

Examples:

```text
POST /api/v1/payments
GET /api/v1/payments/{reference}
POST /api/v1/payments/{reference}/refunds
```

---

## 4. Authentication Models

### Public Merchant API

Uses API keys.

```text
Authorization: Bearer <secret_key>
```

### Merchant Dashboard API

Uses authenticated merchant user session or token depending frontend implementation.

### Internal Admin API

Uses authenticated internal user session or token with RBAC.

---

## 5. Request Format

Use JSON request bodies.

Header:

```http
Content-Type: application/json
Accept: application/json
```

---

## 6. Response Format

Successful response should use consistent structure.

Example:

```json
{
  "data": {},
  "meta": {
    "request_id": "req_xxx"
  }
}
```

For simple APIs, `data` may contain the resource directly.

---

## 7. Error Format

Standard error response:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The given data was invalid.",
    "details": {},
    "request_id": "req_xxx"
  }
}
```

---

## 8. Money Format

Money values are integers in minor units.

Example:

```json
{
  "amount": 10000,
  "currency": "SDG"
}
```

Do not send floating point money amounts.

---

## 9. Pagination

For list endpoints, use cursor pagination where possible.

Example query:

```text
GET /api/v1/payments?limit=50&cursor=xxx
```

Response meta:

```json
{
  "meta": {
    "next_cursor": "xxx",
    "has_more": true
  }
}
```

---

## 10. Request IDs

Every request should have a request ID.

Used for:

- logs
- support
- tracing
- error debugging

Response includes:

```text
request_id
```

---

## 11. Idempotency

Money-affecting creation endpoints require idempotency.

Required for MVP:

```text
POST /api/v1/payments
```

Header:

```http
Idempotency-Key: <unique-key>
```

---

## 12. Webhooks

Webhook events are server-to-server notifications.

Rules:

- signed with HMAC SHA256
- include event ID
- include timestamp
- retry on failure
- track delivery history

---

## 13. API Stability

Public APIs should not break unexpectedly.

Breaking changes require new version.

Non-breaking changes:

- adding optional fields
- adding new event types
- adding new error details

Breaking changes:

- removing fields
- changing meaning of fields
- changing required fields
- changing status semantics

---

## 14. Documentation Requirements

Developer docs must include:

- authentication
- create payment
- payment status
- refunds
- webhooks
- error codes
- idempotency
- test mode
- examples

---

## 15. Critical Rules

1. Public API is versioned from day one.
2. Money uses integer minor units.
3. Currency is explicit.
4. Create Payment requires Idempotency-Key.
5. Errors use standard structure.
6. Webhooks are signed and retryable.
