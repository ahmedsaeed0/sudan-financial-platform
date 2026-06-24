# 08 — Provider Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Provider Domain abstracts external payment infrastructure such as EBS, banks, wallet providers, and future payment rails.

The core platform must not depend directly on one provider implementation.

---

## 2. Responsibilities

The Provider Domain owns:

- PaymentProvider contract
- provider resolver
- MockProvider
- EbsProvider
- provider response normalization
- provider error mapping
- provider attempt tracking
- provider health metrics
- provider timeout handling

The Provider Domain does not own:

- payment business lifecycle
- ledger posting
- merchant approval
- settlement accounting

---

## 3. Key Components

```text
PaymentProvider Contract
ProviderResolver
MockProvider
EbsProvider
ProviderResponse
ProviderError
ProviderAttempt
ProviderHealth
```

---

## 4. Provider Implementations

MVP:

```text
MockProvider
EbsProvider
```

Future:

```text
BankProvider
WalletProvider
CardProvider
QrProvider
```

---

## 5. Provider Resolver

Provider Resolver chooses provider based on:

- API key mode
- merchant configuration
- payment method
- environment
- future routing rules

Initial rule:

```text
TEST -> MockProvider
LIVE -> EbsProvider
```

---

## 6. Normalized Outcomes

Provider-specific responses must be normalized into internal outcomes:

```text
SUCCESS
FAILED
PENDING
TIMEOUT
UNKNOWN
```

The Payment Domain should not deal with raw provider-specific statuses.

---

## 7. Error Mapping

Platform-level provider errors:

```text
PROVIDER_TIMEOUT
PROVIDER_UNAVAILABLE
PROVIDER_DECLINED
INVALID_PROVIDER_RESPONSE
PROVIDER_AUTH_FAILED
UNKNOWN_PROVIDER_ERROR
```

Provider-specific codes should be stored for diagnostics but not leak directly into core domain decisions.

---

## 8. Provider Attempts

Provider calls should be traceable.

Provider attempt concept fields:

```text
id
provider_name
operation
payment_id nullable
refund_id nullable
settlement_id nullable
request_reference
provider_reference
status
normalized_outcome
error_code
latency_ms
created_at
updated_at
```

Operations:

```text
PAYMENT_INITIATE
PAYMENT_VERIFY
REFUND_EXECUTE
SETTLEMENT_EXECUTE
```

---

## 9. Timeout Rule

Timeout is not the same as failure.

If provider times out:

- payment may be PENDING
- refund may remain PROCESSING
- status verification may be scheduled
- reconciliation may be needed

---

## 10. Provider Health

Track:

- latency
- timeout rate
- success rate
- error rate
- unknown status count
- mismatch rate

Provider health can affect future routing and risk decisions.

---

## 11. MockProvider

MockProvider is used for TEST mode.

It should implement the same contract as EbsProvider.

Purpose:

- sandbox testing
- developer integration
- automated tests
- local development

---

## 12. EbsProvider

EbsProvider handles integration with EBS or the first live Sudanese payment infrastructure.

It must stay isolated from Payment core.

EBS-specific request/response mapping belongs inside Provider implementation.

---

## 13. Domain Events

```text
ProviderPaymentStarted
ProviderPaymentSucceeded
ProviderPaymentFailed
ProviderPaymentTimedOut
ProviderRefundStarted
ProviderRefundSucceeded
ProviderRefundFailed
ProviderHealthChanged
ProviderAttemptRecorded
```

---

## 14. Commands

```text
ResolveProvider
InitiateProviderPayment
VerifyProviderPayment
ExecuteProviderRefund
RecordProviderAttempt
CheckProviderHealth
```

---

## 15. Database Tables

MVP recommended:

```text
provider_attempts
```

Future:

```text
providers
provider_health_snapshots
provider_routing_rules
```

---

## 16. Testing Requirements

Test:

- MockProvider success
- MockProvider failure
- provider timeout handling
- provider response normalization
- provider error mapping
- resolver chooses provider correctly
- EbsProvider adapter with mocked HTTP

---

## 17. Critical Rules

1. Payment Domain depends on provider contract, not EBS directly.
2. TEST mode uses MockProvider.
3. LIVE mode initially uses EbsProvider.
4. Provider timeout is not automatic failure.
5. Provider-specific errors must be normalized.
6. Adding a new provider should not rewrite core payment logic.
