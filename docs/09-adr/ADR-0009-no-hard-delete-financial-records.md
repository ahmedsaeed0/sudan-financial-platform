# ADR-0009 — No Hard Delete for Financial Records

Status: Accepted  
Date: 2026-06-24  
Project: Sudan Financial Platform

---

## Context

The platform will store financial and operational records used for:

- support
- settlement
- reconciliation
- audit
- investigation
- reporting

Deleting financial records can break trust and make reconciliation impossible.

---

## Decision

Do not hard delete financial records.

Use statuses, archival fields, reversals, and adjustments instead.

---

## Alternatives Considered

### Hard delete records

Pros:

- simple cleanup
- smaller database

Cons:

- destroys audit history
- breaks reconciliation
- makes investigations impossible
- unsafe for financial systems

### Soft delete everything

Pros:

- preserves rows
- common Laravel pattern

Cons:

- not always semantically correct
- can hide business states
- not enough for financial correction

### Status and immutable financial records

Pros:

- clear lifecycle
- audit-friendly
- reconciliation-friendly
- safer financial behavior

Cons:

- more storage usage
- requires retention policy

---

## Records That Must Not Be Hard Deleted

```text
payments
refunds
settlements
ledger_accounts
journal_entries
ledger_entries
reconciliation_reports
reconciliation_incidents
audit_logs
api_keys
api_key_secrets
merchant_payout_accounts
```

---

## Consequences

Positive:

- strong audit trail
- safer support investigations
- better reconciliation
- better operational trust

Negative:

- larger database over time
- requires archival strategy
- requires access control for old sensitive records

---

## Implementation Notes

Use explicit statuses such as:

```text
CANCELLED
FAILED
REVOKED
DISABLED
ARCHIVED
```

Ledger corrections must use:

```text
REVERSAL
ADJUSTMENT
```

not direct edits or deletes.

---

## Rule

If a record affects money, access, audit, settlement, reconciliation, or support traceability, do not hard delete it.
