# ADR-0004 — Use Double Entry Ledger

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

The platform will process real money movements:

- payments
- fees
- refunds
- settlements
- adjustments

A simple payments table is not enough to safely represent balances.

Calculating merchant balances directly from successful payments is dangerous because it ignores refunds, settlements, fees, adjustments, holds, and reconciliation corrections.

---

## Decision

Use a Double Entry Ledger as the financial source of truth.

Every financial event must produce a balanced journal entry.

```text
Total Debit = Total Credit
```

---

## Alternatives Considered

### Calculate balances from payments table

Pros:

- simple
- fast to build early

Cons:

- incorrect for refunds
- incorrect for settlement
- weak auditability
- hard to reconcile
- unsafe for financial systems

### Single-entry ledger

Pros:

- simpler than double entry

Cons:

- weaker financial validation
- easier to create unbalanced state
- less suitable for settlement/reconciliation

### Double Entry Ledger

Pros:

- strong financial integrity
- supports refunds, settlement, fees, adjustments
- auditable
- easier reconciliation

Cons:

- more complex
- requires careful testing
- requires accounting review

---

## Consequences

Positive:

- safer financial state
- accurate merchant balances
- better settlement correctness
- better audit trail
- better reconciliation

Negative:

- more design work
- more tests required
- developers need accounting model understanding

---

## Implementation Notes

Core tables:

```text
ledger_accounts
journal_entries
ledger_entries
balance_projections
```

Initial accounts:

```text
CUSTOMER_CLEARING
MERCHANT_PAYABLE
PLATFORM_REVENUE
PROVIDER_FEE_ACCOUNT
SWITCH_FEE_ACCOUNT
SETTLEMENT_ACCOUNT
REFUND_RESERVE
ADJUSTMENT_ACCOUNT
```

---

## Rule

Payments are business records. Ledger is financial truth.

No settlement should be calculated from payment rows only.
