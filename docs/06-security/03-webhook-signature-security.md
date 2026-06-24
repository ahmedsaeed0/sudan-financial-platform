# 03 — Webhook Signature Security

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines webhook signature security.

Webhooks notify merchant backend systems about important events such as payment success, refund result, and settlement completion.

Merchant systems must be able to verify that webhook requests came from the platform.

---

## 2. Core Rule

Every webhook must be signed.

Use:

```text
HMAC SHA256
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

## 4. Signing Payload

Recommended signing payload:

```text
timestamp + '.' + raw_request_body
```

Example:

```text
2026-06-24T12:00:00Z.{raw_json_body}
```

---

## 5. Signature Algorithm

```text
signature = HMAC_SHA256(webhook_secret, signing_payload)
```

Header format:

```text
X-Signature: sha256=<hex_signature>
```

---

## 6. Merchant Verification Steps

Merchant backend should:

1. read raw request body
2. read timestamp header
3. build signing payload
4. compute HMAC SHA256 using webhook secret
5. compare using timing-safe comparison
6. verify timestamp freshness
7. deduplicate by event ID
8. process event idempotently

---

## 7. Replay Protection

Use timestamp freshness.

Suggested tolerance:

```text
5 minutes
```

If timestamp is too old, reject webhook unless manually replaying with special rules.

---

## 8. Event Deduplication

Merchant should deduplicate using:

```text
X-Event-Id
```

The platform may retry the same webhook event multiple times.

Merchant must handle duplicate delivery safely.

---

## 9. Secret Storage

Webhook secrets should be protected.

Options:

- encrypted at rest
- stored through secret manager
- stored as secret reference

Do not expose existing webhook secret again after creation if using strong secret model.

---

## 10. Secret Rotation

Future rotation flow:

1. merchant requests rotation
2. new webhook secret generated
3. old and new secrets valid during grace period
4. merchant updates backend
5. old secret expires

---

## 11. Manual Replay

Manual replay must:

- create new delivery attempt
- use valid signature
- be audited
- preserve same event ID or include original event ID depending final API design

Final replay semantics should be defined before public launch.

---

## 12. Failure Responses

Merchant should return 2xx only after successful processing or safe acceptance.

If merchant returns non-2xx, platform retries.

---

## 13. Security Rules

Never include in webhook payload:

- API secret
- webhook secret
- provider credentials
- full sensitive payout details

---

## 14. Critical Rules

1. Sign every webhook.
2. Include timestamp.
3. Include event ID.
4. Merchant must deduplicate events.
5. Use timing-safe signature comparison.
6. Webhook failure does not rollback business event.
