# 06 — Webhook Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Webhook Domain manages reliable server-to-server event delivery from the platform to merchant systems.

Webhook is the source of truth for merchant backend integrations.

Redirect is only user experience.

---

## 2. Responsibilities

The Webhook Domain owns:

- webhook endpoint registration
- webhook endpoint status
- webhook event creation
- webhook delivery attempts
- retry scheduling
- HMAC signing
- delivery result tracking
- manual replay
- dead webhook state

The Webhook Domain does not own:

- payment lifecycle decisions
- refund lifecycle decisions
- settlement lifecycle decisions

It delivers events created by other domains.

---

## 3. Key Entities

```text
WebhookEndpoint
WebhookEvent
WebhookDelivery
WebhookAttempt
WebhookSecret
WebhookDeliveryStatus
```

---

## 4. Webhook Endpoint

A merchant can register webhook endpoints.

Endpoint attributes:

```text
merchant_id
url
secret_reference
events
is_active
created_at
updated_at
```

---

## 5. MVP Events

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

## 6. Delivery Statuses

```text
PENDING
DELIVERED
FAILED
RETRYING
DEAD
```

---

## 7. Delivery Flow

1. Domain event occurs.
2. Webhook event is persisted.
3. Delivery job is queued.
4. Worker loads endpoint and payload.
5. Worker signs payload.
6. Worker sends HTTP request to merchant endpoint.
7. System records response code and result.
8. If success, mark DELIVERED.
9. If failure, schedule retry or mark DEAD.

---

## 8. Retry Strategy

Suggested retry schedule:

```text
immediate
1 minute
5 minutes
15 minutes
1 hour
6 hours
```

After maximum attempts, mark delivery as DEAD.

---

## 9. HMAC Signature

Webhooks must be signed.

Suggested headers:

```text
X-Signature
X-Timestamp
X-Event-Id
```

Signature algorithm:

```text
HMAC SHA256
```

The exact signing payload format must be documented in API docs.

---

## 10. Payload Shape

Recommended payload shape:

```json
{
  "event_id": "...",
  "event_type": "payment.succeeded",
  "created_at": "...",
  "data": {}
}
```

---

## 11. Manual Replay

Manual replay is allowed for failed or dead deliveries.

Rules:

- replay must be audited
- replay should not create a new business event
- replay should create a new delivery attempt

---

## 12. Webhook Failure Rule

Webhook failure must not rollback financial operations.

Example:

PaymentSucceeded remains succeeded even if merchant endpoint is down.

Webhook delivery is retried independently.

---

## 13. Outbox Integration

Critical domain events should be persisted first.

Flow:

```text
PaymentSucceeded
-> Outbox/Webhook Event Stored
-> Worker Delivers Webhook
-> Delivery Result Stored
```

---

## 14. Domain Events

```text
WebhookEndpointRegistered
WebhookEndpointDisabled
WebhookEventCreated
WebhookDeliveryQueued
WebhookDeliveryAttempted
WebhookDelivered
WebhookDeliveryFailed
WebhookRetryScheduled
WebhookMarkedDead
WebhookManuallyReplayed
```

---

## 15. Commands

```text
RegisterWebhookEndpoint
DisableWebhookEndpoint
CreateWebhookEvent
QueueWebhookDelivery
SendWebhook
ScheduleWebhookRetry
MarkWebhookDead
ReplayWebhook
```

---

## 16. Database Tables

```text
webhook_endpoints
webhook_deliveries
```

Possible future:

```text
webhook_events
webhook_attempts
```

---

## 17. Testing Requirements

Test:

- endpoint registration
- payload signing
- successful delivery
- failed delivery
- retry scheduling
- dead state
- manual replay
- signature verification example

---

## 18. Critical Rules

1. Webhooks are asynchronous.
2. Webhook failure does not rollback payment/refund/settlement.
3. Webhooks must be signed.
4. Webhook deliveries must be persisted.
5. Dead webhooks must be visible to merchant/support.
