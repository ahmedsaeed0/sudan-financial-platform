# 05 — Migration Order

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the recommended order for database migrations.

The order matters because financial and merchant tables depend on identity, merchant, and configuration tables.

---

## 2. Migration Philosophy

Do not generate migrations randomly.

A migration should exist only after:

- table purpose is documented
- relationships are known
- indexes are justified
- sensitive data handling is clear
- lifecycle/statuses are known
- audit requirement is clear

---

## 3. Recommended Migration Order

### Phase 1 — Base Infrastructure

```text
create_internal_users_table
create_merchants_table
create_merchant_users_table
```

Reason:

- Internal users are needed for admin operations.
- Merchants are the tenant root.
- Merchant users belong to merchants.

---

### Phase 2 — Merchant Onboarding

```text
create_merchant_documents_table
create_merchant_payout_accounts_table
```

Reason:

- Merchant approval needs documents.
- Settlement needs payout destination.

---

### Phase 3 — API Access

```text
create_api_keys_table
create_api_key_secrets_table
```

Reason:

- Payments created by API need API key ownership.
- Secret rotation requires a separate secrets table.

---

### Phase 4 — Merchant Configuration

```text
create_fee_profiles_table
create_risk_profiles_table
```

Reason:

- Payment creation requires fee calculation.
- Payment creation requires risk checks.

---

### Phase 5 — Payment Core

```text
create_payments_table
create_payment_links_table
create_checkout_sessions_table
create_idempotency_keys_table
```

Reason:

- Payments depend on merchants and API keys.
- Payment links and checkout sessions depend on payments or merchant.
- Idempotency supports Create Payment API.

---

### Phase 6 — Provider Tracking

```text
create_provider_attempts_table
```

Reason:

- Provider calls need traceability.
- Useful for timeout and failure analysis.

This table may be optional in early MVP but is recommended.

---

### Phase 7 — Refunds

```text
create_refunds_table
```

Reason:

- Refunds depend on payments.
- Refunds must be tenant-scoped by merchant.

---

### Phase 8 — Webhooks

```text
create_webhook_endpoints_table
create_webhook_deliveries_table
```

Reason:

- Endpoints belong to merchants.
- Deliveries depend on endpoints.

---

### Phase 9 — Ledger

```text
create_ledger_accounts_table
create_journal_entries_table
create_ledger_entries_table
create_balance_projections_table
```

Reason:

- Ledger accounts are required before ledger entries.
- Journal entries group ledger entries.
- Balance projections depend on ledger entries.

Note:

Ledger can be created before refunds/settlements if implementation starts with ledger earlier.

---

### Phase 10 — Settlement

```text
create_settlements_table
create_settlement_items_table
create_settlement_attempts_table
```

Reason:

- Settlement depends on merchants, payout accounts, and ledger.
- Settlement items and attempts depend on settlement.

---

### Phase 11 — Reconciliation

```text
create_reconciliation_reports_table
create_reconciliation_lines_table
create_reconciliation_incidents_table
```

Reason:

- Reconciliation reports are uploaded/created first.
- Lines and incidents belong to reports.

---

### Phase 12 — Notifications

```text
create_notifications_table
create_email_deliveries_table
```

Reason:

- Notifications are side effects and can be added after core flows.

---

### Phase 13 — Audit and Security Events

```text
create_audit_logs_table
create_security_events_table
```

Reason:

- Audit can be created earlier if needed.
- It should exist before production usage.

---

## 4. Seed Order

Recommended seed order:

1. ledger account codes
2. internal super admin
3. default fee profile template
4. default risk profile template
5. mock provider configuration
6. webhook event types

---

## 5. Production Migration Rules

Before running production migrations:

- take database backup
- review migration SQL
- test in staging
- confirm rollback path if possible
- avoid destructive migration without explicit approval
- monitor queues after deploy

---

## 6. Critical Rule

Migrations are not the design.

Migrations are the implementation of approved design.
