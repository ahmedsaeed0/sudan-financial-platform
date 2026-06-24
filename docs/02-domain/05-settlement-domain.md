# 05 — Settlement Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Settlement Domain manages transferring merchant funds from platform-controlled balances to merchant payout destinations.

Settlement must be based on ledger-derived available balance, not payment table sums.

---

## 2. Responsibilities

The Settlement Domain owns:

- settlement eligibility
- settlement batch generation
- settlement items
- settlement attempts
- settlement execution coordination
- settlement retries
- settlement lifecycle
- settlement reporting

The Settlement Domain collaborates with:

- Merchant Domain for payout destination
- Ledger Domain for available balance and posting
- Provider Domain or manual operations for payout execution
- Reconciliation Domain for matching external reports
- Notification Domain for alerts
- Audit Domain for manual actions

---

## 3. Key Entities

```text
Settlement
SettlementBatch
SettlementItem
SettlementAttempt
Payout
SettlementStatus
```

---

## 4. Settlement States

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

---

## 5. Valid Transitions

```text
PENDING -> PROCESSING
PENDING -> CANCELLED
PROCESSING -> COMPLETED
PROCESSING -> FAILED
FAILED -> PROCESSING
```

`FAILED -> PROCESSING` is allowed only through retry flow.

---

## 6. Settlement Eligibility

Merchant is eligible for settlement if:

1. merchant is APPROVED
2. merchant has verified payout destination
3. available balance meets minimum settlement amount
4. no settlement hold exists
5. settlement schedule allows current run
6. merchant is not suspended from settlement

---

## 7. Balance Source Rule

Settlement amount must come from:

```text
Ledger entries
or
Ledger-derived balance projection
```

Forbidden:

```text
sum successful payments directly
```

---

## 8. Settlement Generation Flow

1. Scheduler or admin starts settlement batch.
2. System identifies eligible merchants.
3. System reads available balance from Ledger.
4. System creates settlement record.
5. System creates settlement items if itemization is enabled.
6. Settlement enters PENDING.

---

## 9. Settlement Processing Flow

1. Worker picks PENDING settlement.
2. Mark PROCESSING.
3. Execute payout through available mechanism.
4. Store external reference if available.
5. Mark COMPLETED or FAILED.
6. On completion, post settlement ledger entries.
7. Queue notification and webhook if configured.

---

## 10. Failure Handling

Failure reasons may include:

```text
PAYOUT_DESTINATION_INVALID
PROVIDER_TIMEOUT
PROVIDER_UNAVAILABLE
TRANSFER_REJECTED
UNKNOWN_ERROR
```

When settlement fails:

1. store failure reason
2. notify operations
3. allow automatic retry if safe
4. allow manual retry with audit
5. keep funds payable until success

---

## 11. Retry Rules

Settlement retry must be controlled.

Rules:

- retry must not create duplicate payout
- retry must be auditable
- provider reference must be checked if available
- uncertain provider result requires reconciliation

---

## 12. Ledger Impact

When settlement completes:

Conceptual posting:

| Account | Direction | Amount |
|---|---|---:|
| MERCHANT_PAYABLE | DEBIT | settlement amount |
| SETTLEMENT_ACCOUNT | CREDIT | settlement amount |

Final accounting treatment should be reviewed with finance.

---

## 13. Settlement Reports

Settlement report should include:

```text
settlement reference
merchant
payout destination
currency
gross amount
total fees
net amount
transaction count
status
processed_at
failure_reason
```

---

## 14. Domain Events

```text
SettlementBatchRequested
SettlementBatchGenerated
SettlementEligibilityCalculated
SettlementCreated
SettlementProcessing
SettlementCompleted
SettlementFailed
SettlementRetryScheduled
SettlementManuallyRetried
SettlementLedgerPostingRequested
SettlementReportGenerated
```

---

## 15. Commands

```text
GenerateSettlementBatch
CreateSettlement
ProcessSettlement
RetrySettlement
CancelSettlement
MarkSettlementCompleted
MarkSettlementFailed
PostSettlementLedger
```

---

## 16. Database Tables

```text
settlements
settlement_items
settlement_attempts
```

Related tables:

```text
merchant_payout_accounts
journal_entries
ledger_entries
reconciliation_reports
reconciliation_incidents
```

---

## 17. Testing Requirements

Test:

- settlement eligibility
- minimum settlement amount
- settlement from ledger balance
- successful settlement
- failed settlement
- retry without duplicate payout
- settlement ledger posting
- manual retry audit

---

## 18. Critical Rules

1. Settlement uses ledger-derived balance.
2. Settlement status changes through state machine.
3. Failed settlement can be retried only through controlled flow.
4. Settlement completion must post ledger entries.
5. Manual settlement actions must be audited.
