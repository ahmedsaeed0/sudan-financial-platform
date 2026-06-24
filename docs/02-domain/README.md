# Domain Documentation

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

This directory defines the core business domains of the Sudan Financial Platform.

Domain documentation explains business rules before implementation.

---

## Documents

| File | Domain |
|---|---|
| [01-merchant-domain.md](./01-merchant-domain.md) | Merchant lifecycle and onboarding |
| [02-payment-domain.md](./02-payment-domain.md) | Payment creation, checkout, lifecycle |
| [03-ledger-domain.md](./03-ledger-domain.md) | Financial source of truth |
| [04-refund-domain.md](./04-refund-domain.md) | Refund workflow and rules |
| [05-settlement-domain.md](./05-settlement-domain.md) | Merchant settlement and payouts |
| [06-webhook-domain.md](./06-webhook-domain.md) | Reliable merchant event delivery |
| [07-risk-domain.md](./07-risk-domain.md) | Risk rules and limits |
| [08-provider-domain.md](./08-provider-domain.md) | External provider abstraction |

---

## Core Rule

No module should be implemented until its domain rules are documented.

Domain documentation leads:

- database design
- API design
- service design
- tests
- operational runbooks

---

## Domain Interaction Principle

Domains should communicate through explicit application services and domain events.

Avoid random cross-module model access.
