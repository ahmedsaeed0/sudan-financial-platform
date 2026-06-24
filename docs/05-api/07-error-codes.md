# 07 — Error Codes

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines standard API error response structure and initial error codes.

Consistent errors improve developer experience and support investigations.

---

## 2. Standard Error Shape

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

## 3. HTTP Status Mapping

| HTTP Status | Meaning |
|---:|---|
| 400 | Bad request |
| 401 | Authentication failed |
| 403 | Authenticated but not allowed |
| 404 | Resource not found |
| 409 | Conflict |
| 422 | Validation error |
| 429 | Rate limited |
| 500 | Internal error |
| 502 | Provider upstream error |
| 503 | Service unavailable |
| 504 | Provider timeout |

---

## 4. General Errors

```text
VALIDATION_ERROR
BAD_REQUEST
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
RATE_LIMITED
INTERNAL_ERROR
SERVICE_UNAVAILABLE
```

---

## 5. Authentication Errors

```text
INVALID_API_KEY
API_KEY_DISABLED
API_KEY_REVOKED
API_KEY_EXPIRED
MERCHANT_NOT_APPROVED
MERCHANT_SUSPENDED
```

---

## 6. Idempotency Errors

```text
IDEMPOTENCY_KEY_REQUIRED
IDEMPOTENCY_CONFLICT
IDEMPOTENCY_REQUEST_PROCESSING
IDEMPOTENCY_RECORD_EXPIRED
```

---

## 7. Payment Errors

```text
PAYMENT_NOT_FOUND
DUPLICATE_ORDER_ID
INVALID_AMOUNT
UNSUPPORTED_CURRENCY
PAYMENT_EXPIRED
PAYMENT_CANCELLED
PAYMENT_ALREADY_SUCCEEDED
PAYMENT_NOT_PAYABLE
```

---

## 8. Refund Errors

```text
REFUND_NOT_FOUND
PAYMENT_NOT_REFUNDABLE
REFUND_ALREADY_EXISTS
PARTIAL_REFUND_NOT_SUPPORTED
REFUND_NOT_APPROVED
REFUND_ALREADY_PROCESSED
```

---

## 9. Provider Errors

```text
PROVIDER_TIMEOUT
PROVIDER_UNAVAILABLE
PROVIDER_DECLINED
PROVIDER_AUTH_FAILED
INVALID_PROVIDER_RESPONSE
UNKNOWN_PROVIDER_ERROR
```

---

## 10. Risk Errors

```text
AMOUNT_LIMIT_EXCEEDED
DAILY_VOLUME_EXCEEDED
RATE_LIMIT_EXCEEDED
FAILED_ATTEMPT_LIMIT_EXCEEDED
IP_RATE_LIMIT_EXCEEDED
RISK_REVIEW_REQUIRED
```

---

## 11. Webhook Errors

```text
WEBHOOK_ENDPOINT_NOT_FOUND
WEBHOOK_ENDPOINT_DISABLED
WEBHOOK_SIGNATURE_INVALID
WEBHOOK_REPLAY_NOT_ALLOWED
```

---

## 12. Ledger Errors

Ledger errors are usually internal and should not expose sensitive internals.

Internal codes:

```text
LEDGER_POSTING_FAILED
LEDGER_ENTRY_UNBALANCED
LEDGER_DUPLICATE_POSTING
LEDGER_ACCOUNT_INACTIVE
```

Public API may return:

```text
INTERNAL_ERROR
```

while internal alert logs detailed ledger error.

---

## 13. Error Response Examples

### Validation Error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The given data was invalid.",
    "details": {
      "amount": ["Amount is required."]
    },
    "request_id": "req_xxx"
  }
}
```

### Duplicate Order ID

```json
{
  "error": {
    "code": "DUPLICATE_ORDER_ID",
    "message": "A payment with this order_id already exists for this merchant.",
    "request_id": "req_xxx"
  }
}
```

### Provider Timeout

```json
{
  "error": {
    "code": "PROVIDER_TIMEOUT",
    "message": "The provider did not respond in time. Payment status may be pending verification.",
    "request_id": "req_xxx"
  }
}
```

---

## 14. Critical Rules

1. Error codes must be stable.
2. Do not expose provider secrets or raw sensitive responses.
3. Always include request_id.
4. Use consistent shape for all API errors.
5. Internal financial errors must alert engineering/operations.
