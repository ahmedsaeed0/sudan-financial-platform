# 06 — Provider Integration Architecture

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how the platform integrates with external payment providers without coupling the core system to one provider.

The first live provider may be EBS, but the architecture must support future providers.

---

## 2. Core Principle

The Payment Domain must not depend directly on EBS or any provider-specific implementation.

Use:

```text
PaymentProvider Contract
```

---

## 3. Architecture

```text
Payment Application Service
        ↓
Provider Resolver
        ↓
PaymentProvider Contract
        ↓
MockProvider / EbsProvider / Future Providers
        ↓
External Payment Infrastructure
```

---

## 4. Provider Contract

The exact interface will be defined during implementation, but conceptually it should support:

```text
initiate payment
verify payment status
execute refund
check provider health
normalize response
```

---

## 5. Provider Resolver

Provider Resolver chooses the correct provider.

Initial rules:

```text
TEST mode -> MockProvider
LIVE mode -> EbsProvider
```

Future rules may include:

- merchant-specific provider
- payment-method provider
- provider health-based routing
- risk-based routing
- cost-based routing

---

## 6. MockProvider

Used for:

- local development
- sandbox
- automated tests
- merchant integration testing

MockProvider must implement the same contract as EbsProvider.

---

## 7. EbsProvider

Used for initial Sudan live provider integration.

EBS-specific logic must stay inside the EbsProvider implementation.

Do not leak raw EBS statuses or response shapes into Payment Domain.

---

## 8. Normalized Provider Outcomes

Every provider response should be normalized into:

```text
SUCCESS
FAILED
PENDING
TIMEOUT
UNKNOWN
```

---

## 9. Provider Errors

Platform-level provider errors:

```text
PROVIDER_TIMEOUT
PROVIDER_UNAVAILABLE
PROVIDER_DECLINED
PROVIDER_AUTH_FAILED
INVALID_PROVIDER_RESPONSE
UNKNOWN_PROVIDER_ERROR
```

Raw provider error codes may be stored for diagnostics.

---

## 10. Provider Attempts

Each provider call should be traceable.

Suggested table:

```text
provider_attempts
```

Core fields:

```text
provider_name
operation
payment_id
refund_id
settlement_id
request_reference
provider_reference
status
normalized_outcome
error_code
latency_ms
created_at
updated_at
```

---

## 11. Timeout Handling

Timeout is not final failure.

For payment:

```text
Provider timeout -> Payment PENDING -> Schedule status verification
```

For refund:

```text
Provider timeout -> Refund remains PROCESSING or unresolved -> Verify later
```

---

## 12. Provider Health

Track:

- latency
- timeout rate
- error rate
- success rate
- unknown status count
- mismatch rate

Provider health can later affect routing and risk.

---

## 13. Testing Strategy

Test provider integration with:

- MockProvider success
- MockProvider failure
- simulated timeout
- EbsProvider mocked HTTP responses
- normalized outcome mapping
- provider resolver tests

---

## 14. Critical Rules

1. Provider-specific code stays in provider implementation.
2. Payment Domain depends on contract only.
3. TEST mode uses MockProvider.
4. Provider timeout is not automatic failure.
5. Provider attempts must be traceable.
6. Adding a provider must not require rewriting payment core.
