# 06 — Webhook Payloads

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines webhook payload structure and signing model.

Webhooks are the source of truth for merchant backend integrations.

Redirect is only customer browser UX.

---

## 2. Delivery Method

Webhook delivery uses HTTP POST.

```http
POST <merchant_webhook_url>
```

Content type:

```http
Content-Type: application/json
```

---

## 3. Required Headers

```http
X-Event-Id: evt_xxx
X-Event-Type: payment.succeeded
X-Timestamp: 2026-06-24T12:00:00Z
X-Signature: sha256=<signature>
```

---

## 4. Signature

Use HMAC SHA256.

Conceptual signing payload:

```text
X-Timestamp + '.' + raw_request_body
```

Signature:

```text
HMAC_SHA256(webhook_secret, signing_payload)
```

Final signing specification must be locked before publishing API docs.

---

## 5. Standard Payload Shape

```json
{
  "id": "evt_xxx",
  "type": "payment.succeeded",
  "created_at": "2026-06-24T12:00:00Z",
  "data": {}
}
```

---

## 6. MVP Event Types

```text
payment.created
payment.processing
payment.succeeded
payment.failed
payment.expired
refund.succeeded
refund.failed
settlement.completed
```

---

## 7. payment.succeeded Payload

```json
{
  "id": "evt_123",
  "type": "payment.succeeded",
  "created_at": "2026-06-24T12:05:00Z",
  "data": {
    "payment": {
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
      "paid_at": "2026-06-24T12:05:00Z"
    }
  }
}
```

---

## 8. payment.failed Payload

```json
{
  "id": "evt_124",
  "type": "payment.failed",
  "created_at": "2026-06-24T12:06:00Z",
  "data": {
    "payment": {
      "reference": "PAY_20260624_ABC123",
      "order_id": "ORDER-10001",
      "status": "FAILED",
      "amount": 10000,
      "currency": "SDG",
      "failure_reason": "PROVIDER_DECLINED",
      "failed_at": "2026-06-24T12:06:00Z"
    }
  }
}
```

---

## 9. refund.succeeded Payload

```json
{
  "id": "evt_200",
  "type": "refund.succeeded",
  "created_at": "2026-06-24T13:10:00Z",
  "data": {
    "refund": {
      "id": "uuidv7",
      "payment_reference": "PAY_20260624_ABC123",
      "amount": 10000,
      "currency": "SDG",
      "status": "SUCCEEDED",
      "processed_at": "2026-06-24T13:10:00Z"
    }
  }
}
```

---

## 10. settlement.completed Payload

```json
{
  "id": "evt_300",
  "type": "settlement.completed",
  "created_at": "2026-06-24T18:00:00Z",
  "data": {
    "settlement": {
      "reference": "SET_20260624_ABC123",
      "status": "COMPLETED",
      "currency": "SDG",
      "net_amount": 983000,
      "transactions_count": 120,
      "processed_at": "2026-06-24T18:00:00Z"
    }
  }
}
```

---

## 11. Retry Behavior

Webhook delivery is retried on non-2xx response or timeout.

Suggested schedule:

```text
immediate
1 minute
5 minutes
15 minutes
1 hour
6 hours
then DEAD
```

---

## 12. Merchant Handling Rules

Merchant system should:

1. verify signature
2. verify timestamp freshness
3. deduplicate using event id
4. process event idempotently
5. return 2xx on success

---

## 13. Security Rules

- never include API secrets
- never include webhook secret
- sign every payload
- include timestamp
- include event id
- support event replay safely

---

## 14. Critical Rules

1. Webhooks are signed.
2. Webhooks are retried.
3. Merchant must deduplicate by event id.
4. Webhook failure does not rollback payment success.
5. Payload shape must remain backward compatible within API version.
