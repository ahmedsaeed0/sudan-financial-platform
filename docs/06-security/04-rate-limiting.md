# 04 — Rate Limiting

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines rate limiting strategy for the platform.

Rate limiting protects the system from abuse, accidental overload, brute force behavior, and provider pressure.

---

## 2. Rate Limiting Layers

Apply rate limiting at multiple layers:

```text
Cloudflare / Edge
Application
API Key
Merchant
IP Address
Endpoint
Risk Profile
```

---

## 3. Edge Rate Limiting

Cloudflare may protect against:

- obvious abuse
- bot traffic
- high request bursts
- DDoS patterns

Edge rate limiting is useful but not enough.

Application must still enforce business-aware limits.

---

## 4. API Key Limits

Apply limits per API key.

Example dimensions:

```text
api_key_id
endpoint
minute
hour
```

Useful for:

- preventing one key from overloading API
- isolating abusive integrations
- debugging merchant-specific issues

---

## 5. Merchant Limits

Merchant limits are business/risk limits.

Examples:

```text
max_transactions_per_minute
max_daily_volume
max_transaction_amount
max_failed_attempts
```

Merchant risk profile controls these values.

---

## 6. IP Limits

IP limits protect public endpoints from abuse.

Useful for:

- login attempts
- checkout abuse
- repeated failed attempts
- API probing

IP limits should not be the only control because merchants may share infrastructure.

---

## 7. Endpoint Limits

Different endpoints need different limits.

Examples:

```text
POST /api/v1/payments -> stricter
GET /api/v1/payments/{reference} -> moderate
Webhook replay -> strict
Login -> strict
```

---

## 8. Create Payment Rate Limits

Create Payment should check:

- API key limit
- merchant velocity limit
- IP limit
- risk profile daily volume
- idempotency behavior

---

## 9. Failed Attempt Limits

Track failed attempts for:

- API authentication
- payment creation validation
- provider declines if risk-relevant
- checkout attempts

Excessive failures may create risk alerts.

---

## 10. Storage

Redis can be used for rate counters.

But final financial invariants must be protected in PostgreSQL.

Redis rate limiting is operational protection, not financial truth.

---

## 11. Rate Limit Response

HTTP:

```text
429 Too Many Requests
```

Error code:

```text
RATE_LIMITED
```

Response:

```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests. Please retry later.",
    "request_id": "req_xxx"
  }
}
```

---

## 12. Headers

Optional response headers:

```http
X-RateLimit-Limit
X-RateLimit-Remaining
X-RateLimit-Reset
Retry-After
```

---

## 13. Risk Integration

Rate limit violations can create risk alerts.

Examples:

```text
RATE_LIMIT_EXCEEDED
FAILED_ATTEMPT_LIMIT_EXCEEDED
IP_RATE_LIMIT_EXCEEDED
```

---

## 14. Critical Rules

1. Rate limiting must exist at edge and application level.
2. Create Payment needs merchant/API key/IP controls.
3. Redis may support counters, but not financial truth.
4. Repeated violations should create risk signals.
5. High-risk endpoints need stricter limits.
