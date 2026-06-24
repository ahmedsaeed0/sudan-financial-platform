# 06 — Ledger Schema

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the database design for the financial ledger.

The ledger is the financial source of truth for:

- payments
- fees
- refunds
- settlements
- adjustments
- merchant balances

Payments are business records. Ledger entries are financial records.

---

## 2. Core Principle

Use Double Entry Ledger.

Every financial event must produce a balanced journal entry.

```text
Total Debit = Total Credit
```

If a journal entry does not balance, it must not be posted.

---

## 3. Ledger Tables

Core tables:

```text
ledger_accounts
journal_entries
ledger_entries
balance_projections
```

Optional future tables:

```text
ledger_entry_metadata
ledger_adjustments
ledger_reversals
```

---

## 4. ledger_accounts

### Purpose

Defines accounts used in financial postings.

### Core Fields

```text
id
code
name
type
currency
is_active
created_at
updated_at
```

### Account Codes

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

### Account Types

Suggested types:

```text
ASSET
LIABILITY
REVENUE
EXPENSE
CLEARING
```

### Constraints

```text
code unique
```

### Indexes

```text
code
type
currency
is_active
```

---

## 5. journal_entries

### Purpose

Represents one financial event.

Examples:

- payment succeeded
- refund succeeded
- settlement completed
- manual adjustment

### Core Fields

```text
id
reference
source_type
source_id
posting_type
description
status
occurred_at
posted_at
created_at
updated_at
```

### Statuses

```text
DRAFT
POSTED
VOIDED
```

MVP should usually create entries directly as POSTED after validation.

### Source Types

```text
PAYMENT
REFUND
SETTLEMENT
ADJUSTMENT
```

### Posting Types

```text
PAYMENT_SUCCESS
REFUND_SUCCESS
SETTLEMENT_COMPLETED
MANUAL_ADJUSTMENT
REVERSAL
```

### Constraints

```text
reference unique
source_type + source_id + posting_type unique
```

The second constraint prevents duplicate posting for the same financial event.

### Indexes

```text
source_type + source_id
posting_type
posted_at
status
```

---

## 6. ledger_entries

### Purpose

Represents one debit or credit line inside a journal entry.

### Core Fields

```text
id
journal_entry_id
ledger_account_id
merchant_id nullable
direction
amount
currency
created_at
updated_at
```

### Directions

```text
DEBIT
CREDIT
```

### Rules

- amount must be positive.
- currency must match the journal/account context.
- each entry belongs to one journal entry.
- posted entries must not be edited.

### Indexes

```text
journal_entry_id
ledger_account_id
merchant_id
created_at
merchant_id + created_at
ledger_account_id + created_at
```

---

## 7. balance_projections

### Purpose

Stores fast balance projections derived from ledger entries.

This table is optional for MVP but recommended once reporting or settlement queries grow.

### Core Fields

```text
id
merchant_id
currency
pending_balance
available_balance
updated_at
created_at
```

### Rules

- This table is a projection, not the source of truth.
- It can be rebuilt from ledger entries.
- If there is a conflict, ledger entries win.

### Constraints

```text
merchant_id + currency unique
```

---

## 8. Payment Success Posting

Example:

```text
Payment amount: 10000
Gateway fee: 100
Provider fee: 50
Switch fee: 20
Net amount: 9830
Currency: SDG
```

Conceptual posting:

| Account | Direction | Amount |
|---|---|---:|
| CUSTOMER_CLEARING | DEBIT | 10000 |
| MERCHANT_PAYABLE | CREDIT | 9830 |
| PLATFORM_REVENUE | CREDIT | 100 |
| PROVIDER_FEE_ACCOUNT | CREDIT | 50 |
| SWITCH_FEE_ACCOUNT | CREDIT | 20 |

Important:

The final accounting treatment for provider and switch fees must be reviewed with accounting/finance once real contracts are available.

---

## 9. Refund Success Posting

Refund accounting depends on fee policy.

Questions:

- Is gateway fee refunded?
- Is provider fee refunded?
- Is switch fee refunded?
- Does the provider return fees or keep them?

MVP must support explicit refund posting rules, not hidden assumptions.

Conceptual full reversal:

| Account | Direction | Amount |
|---|---|---:|
| MERCHANT_PAYABLE | DEBIT | net amount |
| PLATFORM_REVENUE | DEBIT | gateway fee if refunded |
| PROVIDER_FEE_ACCOUNT | DEBIT | provider fee if refunded |
| SWITCH_FEE_ACCOUNT | DEBIT | switch fee if refunded |
| CUSTOMER_CLEARING | CREDIT | gross amount |

---

## 10. Settlement Completed Posting

Settlement moves merchant payable balance toward payout/settlement account.

Conceptual posting:

| Account | Direction | Amount |
|---|---|---:|
| MERCHANT_PAYABLE | DEBIT | settlement net amount |
| SETTLEMENT_ACCOUNT | CREDIT | settlement net amount |

---

## 11. Adjustment Posting

Manual financial correction must be handled through adjustment entries.

Never edit posted ledger entries directly.

Adjustment must include:

```text
reason
approved_by
source reference
supporting evidence
```

---

## 12. Immutability Rules

After posting:

- journal_entries should not be deleted.
- ledger_entries should not be deleted.
- amounts should not be edited.
- corrections require adjustment or reversal.

---

## 13. Posting Safety Checks

Before posting journal entry:

1. Validate all accounts exist.
2. Validate all accounts are active.
3. Validate all amounts are positive integers.
4. Validate currency.
5. Validate total debit equals total credit.
6. Validate source posting uniqueness.
7. Insert journal entry and ledger entries in one database transaction.

---

## 14. Settlement Balance Rule

Settlement must use ledger-derived available balance.

Forbidden:

```text
Summing successful payments directly.
```

Allowed:

```text
Ledger query or ledger-derived balance projection.
```

---

## 15. Open Accounting Questions

These require business/accounting confirmation:

1. Are provider/switch fees liabilities, expenses, or pass-through accounts?
2. Are provider/switch fees refundable?
3. Are settlements net or gross?
4. Does EBS/bank report include fees separately?
5. How should pending vs available balance be timed?
6. Is there any regulatory requirement for holding settlement funds?

---

## 16. Critical Rule

No production payment is complete until its financial impact is posted or queued for guaranteed ledger posting.

If ledger posting fails, the system must raise a critical incident.
