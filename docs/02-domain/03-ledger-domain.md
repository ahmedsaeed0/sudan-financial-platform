# 03 — Ledger Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Ledger Domain is the financial source of truth for the platform.

It records all money movement using double entry accounting.

Payments, refunds, settlements, and adjustments must post to the ledger.

---

## 2. Responsibilities

The Ledger Domain owns:

- ledger accounts
- journal entries
- ledger entries
- balanced posting validation
- merchant balance derivation
- adjustment entries
- reversal entries
- balance projections

The Ledger Domain does not own:

- payment lifecycle
- refund approval
- settlement execution
- provider communication

---

## 3. Key Entities

```text
LedgerAccount
JournalEntry
LedgerEntry
BalanceProjection
LedgerPosting
LedgerAdjustment
LedgerReversal
```

---

## 4. Core Rule

Every journal entry must balance.

```text
Total Debit = Total Credit
```

If it does not balance, it must not be posted.

---

## 5. Ledger Accounts

Initial account codes:

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

Account types:

```text
ASSET
LIABILITY
REVENUE
EXPENSE
CLEARING
```

---

## 6. Journal Entry

A journal entry represents one financial event.

Examples:

```text
PaymentSucceeded
RefundSucceeded
SettlementCompleted
ManualAdjustmentCreated
```

Core attributes:

```text
reference
source_type
source_id
posting_type
status
occurred_at
posted_at
```

---

## 7. Ledger Entry

A ledger entry is one debit or credit line inside a journal entry.

Core attributes:

```text
journal_entry_id
ledger_account_id
merchant_id nullable
direction
amount
currency
```

Direction:

```text
DEBIT
CREDIT
```

Amounts are always positive integers.

Do not use negative numbers to represent debit/credit.

---

## 8. Posting Types

```text
PAYMENT_SUCCESS
REFUND_SUCCESS
SETTLEMENT_COMPLETED
MANUAL_ADJUSTMENT
REVERSAL
```

---

## 9. Payment Success Posting

Example:

```text
Payment amount: 10000
Gateway fee: 100
Provider fee: 50
Switch fee: 20
Net amount: 9830
Currency: SDG
```

Conceptual ledger:

| Account | Direction | Amount |
|---|---|---:|
| CUSTOMER_CLEARING | DEBIT | 10000 |
| MERCHANT_PAYABLE | CREDIT | 9830 |
| PLATFORM_REVENUE | CREDIT | 100 |
| PROVIDER_FEE_ACCOUNT | CREDIT | 50 |
| SWITCH_FEE_ACCOUNT | CREDIT | 20 |

Accounting treatment must be reviewed with finance when real provider contracts are known.

---

## 10. Refund Posting

Refund posting depends on fee policy.

Questions:

- Is gateway fee refunded?
- Is provider fee refunded?
- Is switch fee refunded?
- Does provider keep fees?

The system must not hide these assumptions.

Refund ledger logic must be explicit and tested.

---

## 11. Settlement Posting

Settlement moves merchant payable amount to settlement/payout account.

Conceptual ledger:

| Account | Direction | Amount |
|---|---|---:|
| MERCHANT_PAYABLE | DEBIT | settlement net amount |
| SETTLEMENT_ACCOUNT | CREDIT | settlement net amount |

---

## 12. Balance Model

Merchant balances:

```text
pending_balance
available_balance
```

Balances must come from:

- ledger entries
- or ledger-derived projections

Never calculate settlement balance from payments table only.

---

## 13. Immutability

Posted ledger records are immutable.

Forbidden:

- edit posted amounts
- delete ledger entries
- delete posted journal entries

Corrections must use:

```text
ADJUSTMENT
REVERSAL
```

---

## 14. Idempotent Posting

A financial source event must be posted once.

Use uniqueness rule:

```text
source_type + source_id + posting_type
```

Examples:

```text
PAYMENT + payment_id + PAYMENT_SUCCESS
REFUND + refund_id + REFUND_SUCCESS
SETTLEMENT + settlement_id + SETTLEMENT_COMPLETED
```

---

## 15. Domain Events

```text
LedgerPostingRequested
JournalEntryPosted
LedgerPostingFailed
BalanceProjectionUpdated
LedgerAdjustmentCreated
LedgerReversalCreated
```

---

## 16. Commands

```text
PostPaymentSuccess
PostRefundSuccess
PostSettlementCompleted
CreateLedgerAdjustment
CreateLedgerReversal
RebuildBalanceProjection
```

---

## 17. Database Tables

```text
ledger_accounts
journal_entries
ledger_entries
balance_projections
```

---

## 18. Testing Requirements

Test:

- journal entry balances
- unbalanced journal rejected
- duplicate source posting rejected
- payment success posting
- refund posting scenarios
- settlement posting
- adjustment posting
- balance projection rebuild

---

## 19. Critical Rules

1. Ledger is financial source of truth.
2. Journal entries must balance.
3. Posted ledger entries are immutable.
4. Corrections use adjustments/reversals.
5. Settlement uses ledger-derived balance.
6. Duplicate source posting must be prevented.
