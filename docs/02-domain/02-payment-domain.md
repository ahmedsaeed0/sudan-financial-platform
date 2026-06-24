# 02 — Payment Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Payment Domain manages payment creation, hosted checkout, payment links, payment lifecycle, idempotency, fee snapshots, customer snapshots, provider initiation, and payment state transitions.

Payment is a business record, not the financial source of truth.

The Ledger is the financial source of truth.

---

## 2. Responsibilities

The Payment Domain owns:

- create payment
- validate payment request
- enforce order_id uniqueness
- enforce idempotency
- create payment reference
- store payment fee snapshot
- store customer snapshot
- create hosted checkout session
- create payment link
- handle payment expiry
- manage payment lifecycle
- coordinate with provider through abstraction

The Payment Domain does not own:

- ledger posting internals
- provider implementation internals
- settlement calculation
- merchant approval workflow

---

## 3. Key Entities

```text
Payment
PaymentLink
CheckoutSession
CustomerSnapshot
IdempotencyRecord
PaymentStatus
```

---

## 4. Payment Fields

Core payment fields:

```text
id
merchant_id
api_key_id
reference
order_id
amount
currency
gateway_fee
provider_fee
switch_fee
total_fee
net_amount
customer_name
customer_email
customer_phone
description
status
provider_name
provider_reference
failure_reason
callback_url
return_url
metadata_json
expires_at
paid_at
failed_at
cancelled_at
ip_address
user_agent
created_at
updated_at
```

---

## 5. Payment States

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

## 6. Valid State Transitions

```text
CREATED -> PENDING
CREATED -> PROCESSING
CREATED -> EXPIRED
CREATED -> CANCELLED
PENDING -> PROCESSING
PENDING -> SUCCEEDED
PENDING -> FAILED
PENDING -> EXPIRED
PROCESSING -> SUCCEEDED
PROCESSING -> FAILED
PROCESSING -> PENDING
SUCCEEDED -> REFUNDED
```

Invalid without explicit recovery flow:

```text
FAILED -> SUCCEEDED
EXPIRED -> SUCCEEDED
CANCELLED -> SUCCEEDED
REFUNDED -> SUCCEEDED
```

---

## 7. Create Payment Flow

1. Receive request.
2. Validate API key.
3. Resolve merchant.
4. Check merchant status.
5. Check Idempotency-Key.
6. Validate amount and currency.
7. Validate order_id uniqueness.
8. Run risk checks.
9. Calculate fees.
10. Store payment with fee snapshot.
11. Create checkout session if needed.
12. Return payment reference and checkout URL.

---

## 8. Idempotency Rule

Create Payment API requires:

```http
Idempotency-Key
```

Idempotency scope:

```text
merchant_id
api_key_id
endpoint
idempotency_key
```

Rules:

- same key + same request returns same response
- same key + different request returns idempotency conflict
- idempotency must be enforced by database uniqueness

---

## 9. Order ID Rule

`order_id` is supplied by the merchant.

Rule:

```text
unique(merchant_id, order_id)
```

Same merchant cannot reuse the same order ID.

Different merchants may use the same order ID.

---

## 10. Currency Rule

MVP supports:

```text
SDG
```

But every payment must store currency separately.

---

## 11. Amount Rule

Amounts must be integer minor units.

Forbidden:

```text
float
double
```

---

## 12. Payment Expiry

Default expiry:

```text
30 minutes
```

Maximum expiry:

```text
7 days
```

Expired payment cannot be paid unless a new payment is created.

---

## 13. Hosted Checkout

Hosted Checkout responsibilities:

- display merchant and amount
- allow customer to complete payment
- handle provider response
- display success/failure/expired result
- redirect customer to return_url

Important:

Redirect is UX only.

Webhook is source of truth for merchant server integration.

---

## 14. Payment Links

Payment links allow merchants to collect payment without direct API integration.

Useful for:

- WhatsApp sellers
- Instagram stores
- small businesses
- agencies
- manual sales teams

Payment links use Hosted Checkout internally.

---

## 15. Provider Timeout Rule

If provider times out:

- do not immediately mark payment as FAILED
- mark as PENDING or keep unresolved state
- schedule provider status verification
- include in reconciliation if unresolved

---

## 16. Payment Success Effects

When `PaymentSucceeded`:

1. store provider reference
2. set paid_at
3. post ledger entries or create guaranteed ledger posting job
4. queue webhook event
5. queue notification
6. update reporting projection
7. make amount eligible for settlement according to settlement rules

---

## 17. Payment Failure Effects

When `PaymentFailed`:

1. store failure reason
2. set failed_at
3. queue webhook event
4. update risk counters if needed
5. update reporting projection

---

## 18. Domain Events

```text
PaymentCreateRequested
IdempotencyKeyChecked
PaymentCreated
CheckoutSessionCreated
PaymentProcessing
ProviderPaymentSucceeded
ProviderPaymentFailed
ProviderPaymentTimedOut
PaymentSucceeded
PaymentFailed
PaymentExpired
PaymentCancelled
PaymentRefunded
```

---

## 19. Commands

```text
CreatePayment
CreatePaymentLink
CreateCheckoutSession
StartPaymentProcessing
MarkPaymentSucceeded
MarkPaymentFailed
ExpirePayment
CancelPayment
MarkPaymentRefunded
```

---

## 20. Database Tables

```text
payments
payment_links
checkout_sessions
idempotency_keys
```

Related tables:

```text
api_keys
merchants
fee_profiles
risk_profiles
journal_entries
ledger_entries
webhook_deliveries
```

---

## 21. Testing Requirements

Test:

- create payment success
- duplicate order_id rejection
- idempotent retry returns same response
- idempotency conflict on changed payload
- invalid state transitions
- payment expiry
- provider timeout behavior
- successful payment queues ledger/webhook effects

---

## 22. Critical Rules

1. Create Payment requires Idempotency-Key.
2. Payment status changes through state machine.
3. Payments do not represent balances.
4. Ledger represents financial truth.
5. Provider timeout is not automatically failure.
6. Redirect is UX; webhook is truth.
