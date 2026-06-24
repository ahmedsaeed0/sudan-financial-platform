# 03 — Create Payment API

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the Create Payment API contract.

This is the most important public API in the MVP.

---

## 2. Endpoint

```http
POST /api/v1/payments
```

---

## 3. Authentication

Required:

```http
Authorization: Bearer <secret_key>
```

The secret key must be active.

LIVE secret key requires merchant to be approved.

---

## 4. Required Headers

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer <secret_key>
Idempotency-Key: <unique-key>
```

---

## 5. Request Body

```json
{
  "amount": 10000,
  "currency": "SDG",
  "order_id": "ORDER-10001",
  "description": "Order payment",
  "customer": {
    "name": "Customer Name",
    "email": "customer@example.com",
    "phone": "+249900000000"
  },
  "return_url": "https://merchant.com/payment-return",
  "callback_url": "https://merchant.com/webhooks/payment",
  "expires_at": null,
  "metadata": {
    "cart_id": "cart_123"
  }
}
```

---

## 6. Required Fields

```text
amount
currency
order_id
return_url
```

Customer fields may be required depending risk/provider requirements.

Recommended MVP customer fields:

```text
customer.name
customer.phone
```

---

## 7. Amount Rules

`amount` must be an integer minor unit.

Forbidden:

```json
{ "amount": 100.50 }
```

Allowed:

```json
{ "amount": 10050 }
```

Exact SDG minor unit policy must be confirmed.

---

## 8. Currency Rules

MVP supports only:

```text
SDG
```

If unsupported currency is provided, return:

```text
UNSUPPORTED_CURRENCY
```

---

## 9. order_id Rule

`order_id` is merchant-defined.

Constraint:

```text
unique(merchant_id, order_id)
```

If duplicate:

```text
DUPLICATE_ORDER_ID
```

---

## 10. Idempotency Behavior

Create Payment requires:

```http
Idempotency-Key
```

Rules:

- same key + same request returns same response
- same key + different request returns `IDEMPOTENCY_CONFLICT`
- no key returns `IDEMPOTENCY_KEY_REQUIRED`

---

## 11. Payment Expiry

Default expiry:

```text
30 minutes
```

Maximum expiry:

```text
7 days
```

If `expires_at` is null, use default.

---

## 12. Successful Response

HTTP status:

```http
201 Created
```

Response:

```json
{
  "data": {
    "id": "uuidv7",
    "reference": "PAY_20260624_ABC123",
    "order_id": "ORDER-10001",
    "status": "CREATED",
    "amount": 10000,
    "currency": "SDG",
    "fees": {
      "gateway_fee": 100,
      "provider_fee": 50,
      "switch_fee": 20,
      "total_fee": 170,
      "net_amount": 9830
    },
    "checkout_url": "https://pay.example.com/checkout/session_xxx",
    "expires_at": "2026-06-24T12:30:00Z",
    "created_at": "2026-06-24T12:00:00Z"
  },
  "meta": {
    "request_id": "req_xxx"
  }
}
```

---

## 13. Error Responses

### Missing Idempotency Key

```json
{
  "error": {
    "code": "IDEMPOTENCY_KEY_REQUIRED",
    "message": "Idempotency-Key header is required.",
    "request_id": "req_xxx"
  }
}
```

### Idempotency Conflict

```json
{
  "error": {
    "code": "IDEMPOTENCY_CONFLICT",
    "message": "The same idempotency key was used with a different request payload.",
    "request_id": "req_xxx"
  }
}
```

### Merchant Suspended

```json
{
  "error": {
    "code": "MERCHANT_SUSPENDED",
    "message": "Merchant is suspended and cannot create live payments.",
    "request_id": "req_xxx"
  }
}
```

---

## 14. Internal Flow

```text
Receive Request
    ↓
Authenticate API Key
    ↓
Resolve Merchant
    ↓
Validate Merchant Status
    ↓
Check Idempotency-Key
    ↓
Validate Payload
    ↓
Run Risk Checks
    ↓
Calculate Fees
    ↓
Create Payment
    ↓
Create Checkout Session
    ↓
Store Idempotency Response
    ↓
Return Response
```

---

## 15. Side Effects

Create Payment may emit:

```text
PaymentCreated
CheckoutSessionCreated
```

Do not call external provider during Create Payment unless product flow explicitly requires it.

Provider call usually happens when customer submits checkout.

---

## 16. Testing Requirements

Test:

- create payment success
- missing auth
- disabled API key
- suspended merchant
- missing idempotency key
- idempotent retry
- idempotency conflict
- duplicate order_id
- invalid amount
- unsupported currency
- expiry default
- expiry max validation

---

## 17. Critical Rules

1. Create Payment requires Idempotency-Key.
2. order_id is unique per merchant.
3. Money is integer minor units.
4. Fees are snapshotted on payment.
5. Response must include checkout_url.
6. Payment creation does not equal payment success.
