# 09 — Idempotency Storage

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how API idempotency is stored and enforced.

Idempotency prevents duplicate side effects caused by:

- client retries
- network failures
- double clicks
- mobile app retry behavior
- gateway timeout retry

---

## 2. Core Rule

Create Payment API must require:

```http
Idempotency-Key: <unique-key>
```

Same request with same idempotency key should return the same result.

Same key with different request body should return conflict.

---

## 3. Table: idempotency_keys

### Purpose

Stores idempotency records for API requests.

### Core Fields

```text
id
merchant_id
api_key_id
idempotency_key
endpoint
request_hash
response_json
status
resource_type
resource_id
locked_until
expires_at
created_at
updated_at
```

---

## 4. Statuses

```text
PROCESSING
COMPLETED
FAILED
EXPIRED
```

### PROCESSING

The request is currently being handled.

### COMPLETED

The request completed and response can be reused.

### FAILED

The request failed in a controlled way.

### EXPIRED

The idempotency record is no longer valid for retry.

---

## 5. Unique Constraint

```text
unique(merchant_id, api_key_id, endpoint, idempotency_key)
```

Reason:

Same key can be safely scoped by merchant, API key, and endpoint.

---

## 6. Request Hash

Store a hash of the normalized request body.

Purpose:

- detect if the same key is reused with different payload
- return idempotency conflict

Conflict rule:

```text
same key + different request_hash = IDEMPOTENCY_CONFLICT
```

---

## 7. Response Storage

Store the response JSON for completed requests.

Purpose:

- repeated request returns same response
- client can safely retry after timeout

---

## 8. Resource Reference

Store created resource reference:

```text
resource_type
resource_id
```

Example:

```text
PAYMENT
payment_id
```

Useful for debugging and recovery.

---

## 9. Processing Lock

`locked_until` can be used to handle stuck PROCESSING records.

Scenario:

- request starts
- idempotency row created as PROCESSING
- server crashes before completion

Recovery policy:

- if locked_until has passed, system can retry or mark failed depending operation safety

---

## 10. Expiry

Idempotency records should expire after a retention period.

Suggested MVP:

```text
24 to 72 hours
```

For payment systems, longer retention may be safer.

Final policy should be reviewed.

---

## 11. Create Payment Flow

1. Receive request.
2. Validate Idempotency-Key exists.
3. Compute request_hash.
4. Try to create idempotency row as PROCESSING.
5. If row already exists:
   - if request_hash differs, return conflict.
   - if status is COMPLETED, return stored response.
   - if status is PROCESSING, return retryable response or wait depending policy.
6. Process payment creation.
7. Store resource_type/resource_id.
8. Store response_json.
9. Mark COMPLETED.

---

## 12. Database Transaction Rule

Payment creation and idempotency completion should be coordinated carefully.

Recommended:

- create idempotency row first
- create payment in database transaction
- update idempotency row with response

If payment is created but response update fails, recovery job should reconcile the idempotency record.

---

## 13. What Idempotency Is Not

Idempotency is not concurrency control.

It prevents duplicate effects from repeated same request.

It does not replace:

- row locks
- unique constraints
- ledger posting uniqueness
- settlement locking

---

## 14. Endpoints Requiring Idempotency

MVP required:

```text
POST /api/v1/payments
```

Future candidates:

```text
POST /api/v1/refunds
POST /api/v1/payment-links
POST /api/v1/settlements/manual
```

---

## 15. Critical Rule

Idempotency must be enforced with database uniqueness, not only application memory or Redis.

Redis can help with short locks, but PostgreSQL must protect the final invariant.
