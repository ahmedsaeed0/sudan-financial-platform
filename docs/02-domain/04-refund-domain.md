# 04 — Refund Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Refund Domain manages refund requests, approval workflow, execution, provider refund calls, refund lifecycle, and refund ledger impact.

Refunds affect money, so they must be controlled, audited, and ledger-posted.

---

## 2. MVP Decision

MVP supports full refund only.

Merchant can request a refund.

Internal admin must approve or reject the refund.

Merchant cannot directly execute refund in MVP.

---

## 3. Responsibilities

The Refund Domain owns:

- refund request creation
- refund eligibility checks
- refund approval/rejection
- refund execution coordination
- refund status transitions
- refund failure handling
- refund retry policy

The Refund Domain collaborates with:

- Payment Domain to confirm payment eligibility
- Provider Domain to execute refund
- Ledger Domain to post financial impact
- Webhook Domain to notify merchant
- Notification Domain to notify users
- Audit Domain to record sensitive actions

---

## 4. Key Entities

```text
Refund
RefundRequest
RefundApproval
RefundExecution
RefundStatus
```

---

## 5. Refund States

```text
REQUESTED
PENDING_APPROVAL
APPROVED
REJECTED
PROCESSING
SUCCEEDED
FAILED
```

---

## 6. Valid Transitions

```text
REQUESTED -> PENDING_APPROVAL
PENDING_APPROVAL -> APPROVED
PENDING_APPROVAL -> REJECTED
APPROVED -> PROCESSING
PROCESSING -> SUCCEEDED
PROCESSING -> FAILED
FAILED -> PROCESSING
```

`FAILED -> PROCESSING` is only allowed through audited admin retry.

---

## 7. Refund Eligibility

A payment can be refunded if:

1. payment status is SUCCEEDED
2. payment has not already been refunded in MVP
3. refund amount equals full payment amount in MVP
4. merchant is authorized to request refund
5. internal admin approves request

---

## 8. Refund Request Flow

1. Merchant user opens successful payment.
2. Merchant submits refund request with reason.
3. System creates refund as REQUESTED or PENDING_APPROVAL.
4. Internal operations/admin reviews request.
5. Admin approves or rejects.

---

## 9. Approval Flow

When approved:

1. audit approval
2. mark refund APPROVED
3. dispatch refund execution
4. mark PROCESSING
5. call provider through PaymentProvider contract
6. mark SUCCEEDED or FAILED

When rejected:

1. audit rejection
2. store rejection reason
3. notify merchant

---

## 10. Refund Success Effects

When `RefundSucceeded`:

1. store provider reference
2. mark refund SUCCEEDED
3. update payment status to REFUNDED
4. post refund ledger entries
5. queue webhook event
6. queue notification
7. update reporting projection

---

## 11. Refund Failure Effects

When `RefundFailed`:

1. store failure reason
2. notify operations
3. allow audited retry if safe
4. keep original payment state consistent

---

## 12. Provider Timeout Rule

If provider refund call times out:

- do not blindly mark refund as FAILED
- mark as PROCESSING or unresolved depending policy
- schedule provider status verification if available
- include in reconciliation if needed

---

## 13. Ledger Impact

Refund must post ledger entries.

Exact accounting depends on fee policy:

- gateway fee refunded or retained
- provider fee refunded or retained
- switch fee refunded or retained

These rules must be explicit before production.

---

## 14. Domain Events

```text
RefundRequested
RefundReviewStarted
RefundApproved
RefundRejected
RefundProcessing
ProviderRefundSucceeded
ProviderRefundFailed
ProviderRefundTimedOut
RefundSucceeded
RefundFailed
RefundLedgerPostingRequested
RefundWebhookQueued
```

---

## 15. Commands

```text
RequestRefund
ApproveRefund
RejectRefund
ExecuteRefund
MarkRefundSucceeded
MarkRefundFailed
RetryRefund
PostRefundLedger
```

---

## 16. Database Tables

```text
refunds
```

Related tables:

```text
payments
journal_entries
ledger_entries
webhook_deliveries
audit_logs
```

---

## 17. Testing Requirements

Test:

- refund request for succeeded payment
- reject refund for non-succeeded payment
- prevent duplicate refund in MVP
- approval flow
- rejection flow
- provider failure
- provider timeout
- refund ledger posting
- audit logs
- webhook event

---

## 18. Critical Rules

1. MVP supports full refund only.
2. Refund requires internal approval.
3. Refund status changes through state machine.
4. Refund approval must be audited.
5. Refund success must post ledger entries.
6. Provider timeout is not automatically final failure.
