# ADR-0005 — Use Provider Abstraction

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

The MVP will likely integrate with EBS or local Sudanese payment infrastructure.

However, the platform should not become tightly coupled to one provider.

Future providers may include:

- another bank
- wallet provider
- card processor
- QR provider
- direct bank API

---

## Decision

Use a PaymentProvider abstraction.

The Payment domain depends on a provider contract, not on EBS directly.

---

## Alternatives Considered

### Direct EBS integration inside Payment service

Pros:

- faster initial implementation
- fewer files/classes

Cons:

- tightly coupled
- hard to test
- hard to add MockProvider
- hard to add future providers
- provider-specific errors leak into core domain

### Provider abstraction

Pros:

- supports TEST and LIVE cleanly
- supports MockProvider
- isolates provider-specific logic
- easier to add new providers
- cleaner payment domain

Cons:

- more initial design
- requires normalized provider responses

---

## Consequences

Positive:

- better testing
- cleaner sandbox
- easier provider expansion
- easier provider health tracking

Negative:

- requires contract design
- requires error normalization
- requires provider resolver

---

## Implementation Notes

Provider implementations:

```text
MockProvider
EbsProvider
FutureBankProvider
FutureWalletProvider
```

Provider resolver rules:

```text
TEST API key -> MockProvider
LIVE API key -> EbsProvider initially
```

Provider response should normalize to:

```text
SUCCESS
FAILED
PENDING
TIMEOUT
UNKNOWN
```

---

## Rule

Adding a provider should not require rewriting Payment, Refund, Ledger, Webhook, or Settlement core logic.
