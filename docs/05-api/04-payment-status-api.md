# 04 — Payment Status API

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines APIs for retrieving payment details and current payment status.

Merchants must be able to query payment status in addition to receiving webhooks.

Webhook is still the primary server-to-server notification mechanism.

---

## 2. Endpoint

```http
GET /api/v1/payments/{reference}
```

---

## 3. Authentication

Required:

```http
Authorization: Bearer <secret_key>
```

Merchant can only access its own payments.

---

## 4. Path Parameters

```text
reference
```

Example:

```text
PAY_20260624_ABC123
```

---

## 5. Successful Response

```json
{
  "data": {
    "id": "uuidv7",
    "reference": "PAY_20260624_ABC123",
    "order_id": "ORDER-10001",
    "status": "SUCCEEDED",
    "amount": 10000,
    "currency": "SDG",
    "fees": {
      "gateway_fee": 100,
      "provider_fee": 50,
      "switch_fee": 20,
      "total_fee": 170,
      "net_amount": 9830
    },
    "customer": {
      "name": "Customer Name",
      "email": "customer@example.com",
      "phone": "+249900000000"
    },
    "provider": {
      "name": "EBS",
      "reference": "EBS_REF_123"
    },
    "created_at": "2026-06-24T12:00:00Z",
    "expires_at": "2026-06-24T12:30:00Z",
    "paid_at": "2026-06-24T12:05:00Z",
    "failed_at": null
  },
  "meta": {
    "request_id": "req_xxx"
  }
}
```

---

## 6. Payment Statuses

```text
CREATED
PENDING
PROCESSING
SUCCEEDED
FAILED
EXPIRED
CANCELLED
REFUNDED
```

---

## 7. Status Meaning

### CREATED

Payment record exists but customer has not started processing.

### PENDING

Payment is waiting for provider confirmation or unresolved status.

### PROCESSING

Payment is actively being processed.

### SUCCEEDED

Payment succeeded.

Ledger posting should exist or be guaranteed.

### FAILED

Payment failed.

### EXPIRED

Payment expired before completion.

### CANCELLED

Payment was cancelled before success.

### REFUNDED

Payment was successfully refunded.

---

## 8. Error: Not Found

If payment does not exist for merchant:

```json
{
  "error": {
    "code": "PAYMENT_NOT_FOUND",
    "message": "Payment was not found.",
    "request_id": "req_xxx"
  }
}
```

Use 404.

---

## 9. Error: Unauthorized

If API key is invalid:

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication failed.",
    "request_id": "req_xxx"
  }
}
```

---

## 10. Merchant Isolation

A merchant must never retrieve another merchant payment.

Filtering must include:

```text
merchant_id
reference
```

---

## 11. List Payments API

Future or MVP dashboard endpoint:

```http
GET /api/v1/payments
```

Query parameters:

```text
status
from
to
limit
cursor
```

---

## 12. Testing Requirements

Test:

- retrieve payment success
- not found
- another merchant payment blocked
- invalid API key
- status values
- provider reference visibility
- fee breakdown visibility

---

## 13. Critical Rules

1. Merchant can only access its own payments.
2. Payment status API does not replace webhook.
3. Provider timeout may show PENDING, not FAILED.
4. Fee snapshot must match original payment.
