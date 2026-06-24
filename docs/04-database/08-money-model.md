# 08 — Money Model

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how money is represented in the database and application.

Money bugs are dangerous in financial systems.

---

## 2. Core Rules

1. Never use float.
2. Never use double.
3. Store money as integer minor units.
4. Store currency separately.
5. Use explicit value objects in application code.
6. Snapshot fees at payment time.
7. Do not recalculate old payment fees from current fee profile.

---

## 3. Database Representation

Money fields should be stored as integer values.

Examples:

```text
amount BIGINT
gateway_fee BIGINT
provider_fee BIGINT
switch_fee BIGINT
total_fee BIGINT
net_amount BIGINT
pending_balance BIGINT
available_balance BIGINT
```

Currency:

```text
currency CHAR(3)
```

MVP currency:

```text
SDG
```

---

## 4. Why Integer Minor Units

Using integers prevents floating point rounding errors.

Bad:

```text
100.10 as FLOAT
```

Good:

```text
10010 as integer minor units
```

Exact minor unit scaling for SDG must be confirmed.

---

## 5. Application Value Object

The application should use a Money value object.

Concept:

```text
Money
- amount
- currency
```

Rules:

- cannot add different currencies
- cannot create negative amount unless explicitly allowed
- formats for display only at UI boundary
- stores integer amount internally

---

## 6. Payment Fee Snapshot

Payment must store:

```text
amount
currency
gateway_fee
provider_fee
switch_fee
total_fee
net_amount
```

Reason:

If merchant fee profile changes later, historical payments must remain correct.

---

## 7. Fee Formula

Conceptual formula:

```text
total_fee = gateway_fee + provider_fee + switch_fee
net_amount = gross_amount - total_fee
```

Where:

```text
gross_amount = payment.amount
```

---

## 8. Rounding Strategy

Percentage fees must define a rounding rule.

Recommended:

- calculate using integer-safe decimal logic
- round to nearest minor unit or defined business rule
- document the rule
- test edge cases

Open question:

Exact rounding policy for Sudanese payment contracts must be confirmed.

---

## 9. Multi-Currency Future

MVP supports only SDG.

Future support may require:

- currency table
- currency minor unit definition
- exchange rates
- FX ledger accounts
- settlement currency
- merchant default currency

Do not implement FX in MVP.

But do not hardcode the schema in a way that blocks it.

---

## 10. Negative Amounts

Payment amounts should be positive.

Ledger entries should use positive amounts with direction:

```text
DEBIT
CREDIT
```

Do not represent debit/credit using negative numbers.

---

## 11. Balance Model

Merchant balances:

```text
pending_balance
available_balance
```

These must be derived from ledger entries or ledger projections.

Do not derive balances from payments table.

---

## 12. Display Formatting

Formatting belongs at UI/API presentation layer.

Database stores integer amount and currency.

Example API display can include:

```json
{
  "amount": 10000,
  "currency": "SDG",
  "display_amount": "100.00 SDG"
}
```

Only if minor unit scale is defined.

---

## 13. Testing Requirements

Money tests must cover:

- addition
- subtraction
- fee calculation
- percentage rounding
- net amount calculation
- different currency rejection
- zero amount rejection
- negative amount rejection where inappropriate

---

## 14. Critical Rule

Money is a domain concept, not just a database number.

Every money operation must be explicit, typed, and tested.
