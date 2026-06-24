# 05 — Refund API

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the Refund Request API for MVP.

MVP supports full refund only and requires internal admin approval before execution.

---

## 2. Endpoint

```http
POST /api/v1/payments/{reference}/refunds
```

---

## 3. Authentication

Required:

```http
Authorization: Bearer <secret_key>
```

Merchant can only request refund for its own payment.

---

## 4. MVP Behavior

MVP behavior:

- merchant requests refund
- refund goes to internal approval queue
- admin approves or rejects
- approved refund is executed through provider
- refund result triggers webhook

Merchant cannot directly execute refund in MVP.

---

## 5. Request Body

```json
{
  "reason": "Customer requested cancellation"
}
```

MVP full refund does not require amount.

If amount is provided in MVP, it should either be rejected or ignored based on final product decision.

Recommended: reject amount for MVP to avoid confusion.

---

## 6. Successful Response

HTTP status:

```http
202 Accepted
```

Response:

```json
{
  "data": {
    "id": "uuidv7",
    "payment_reference": "PAY_20260624_ABC123",
    "amount": 10000,
    "currency": "SDG",
    "status": "PENDING_APPROVAL",
    "reason": "Customer requested cancellation",
    "requested_at": "2026-06-24T13:00:00Z"
  },
  "meta": {
    "request_id": "req_xxx"
  }
}
```

---

## 7. Refund Statuses

```text
REQUESTED
PENDING_APPROVAL
APPROVED
REJECTED
PROCESSING
SUCCEEDED
FAILED
```

---

## 8. Eligibility Rules

A refund can be requested only if:

1. payment belongs to merchant
2. payment status is SUCCEEDED
3. payment is not already REFUNDED
4. no active refund request exists for the payment in MVP

---

## 9. Error Cases

### Payment Not Found

```text
PAYMENT_NOT_FOUND
```

### Payment Not Refundable

```text
PAYMENT_NOT_REFUNDABLE
```

### Refund Already Exists

```text
REFUND_ALREADY_EXISTS
```

### Partial Refund Not Supported

```text
PARTIAL_REFUND_NOT_SUPPORTED
```

---

## 10. Internal Admin Flow

Admin endpoints will be defined separately.

Conceptual internal endpoints:

```http
POST /admin/refunds/{id}/approve
POST /admin/refunds/{id}/reject
POST /admin/refunds/{id}/retry
```

Admin approval must be audited.

---

## 11. Webhook Events

Refund may emit:

```text
refund.succeeded
refund.failed
```

Possible future:

```text
refund.requested
refund.rejected
```

---

## 12. Testing Requirements

Test:

- successful refund request
- refund request for non-succeeded payment rejected
- duplicate refund request rejected
- another merchant payment blocked
- partial refund rejected in MVP
- admin approval creates provider execution
- refund success posts ledger entries

---

## 13. Critical Rules

1. MVP supports full refund only.
2. Refund requires internal approval.
3. Refund must be audited.
4. Refund success must post ledger entries.
5. Refund result must notify merchant through webhook.
