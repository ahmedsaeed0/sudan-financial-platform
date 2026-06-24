# 08 — Idempotency Behavior

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the API behavior for idempotent requests.

Idempotency allows merchants to safely retry requests without creating duplicate effects.

---

## 2. Required Header

```http
Idempotency-Key: <unique-key>
```

Required for:

```text
POST /api/v1/payments
```

Future candidates:

```text
POST /api/v1/refunds
POST /api/v1/payment-links
```

---

## 3. Scope

Idempotency key is scoped by:

```text
merchant_id
api_key_id
endpoint
idempotency_key
```

This means two merchants can use the same key safely.

---

## 4. Same Request Retry

If the same key is used with the same request payload:

Return the original response.

Example:

```text
First request -> 201 Created
Retry request -> same 201 response body
```

---

## 5. Different Payload Conflict

If same key is used with different request body:

Return:

```text
409 Conflict
```

Error code:

```text
IDEMPOTENCY_CONFLICT
```

---

## 6. Processing State

If the first request is still processing:

Recommended response:

```text
409 Conflict or 202 Accepted
```

Error/code option:

```text
IDEMPOTENCY_REQUEST_PROCESSING
```

Final behavior should be decided before API publication.

---

## 7. Request Hash

The platform stores hash of normalized request body.

Purpose:

- detect same payload
- detect conflict
- prevent accidental key reuse

---

## 8. Stored Response

After successful processing, store response JSON.

This enables retry to return same result.

---

## 9. Expiry

Idempotency records should expire after retention period.

Suggested:

```text
24 to 72 hours minimum
```

For payments, longer retention may be safer.

---

## 10. Missing Key

If required endpoint is called without key:

HTTP:

```text
400 Bad Request
```

Error:

```text
IDEMPOTENCY_KEY_REQUIRED
```

---

## 11. What Idempotency Does Not Replace

Idempotency does not replace:

- database unique constraints
- row locks
- ledger duplicate posting protection
- state machines
- settlement payout protection

---

## 12. Critical Rules

1. Idempotency must be durable.
2. Use PostgreSQL uniqueness, not only Redis.
3. Same key + same payload returns same response.
4. Same key + different payload returns conflict.
5. Create Payment requires Idempotency-Key.
