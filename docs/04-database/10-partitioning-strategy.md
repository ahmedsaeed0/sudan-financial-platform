# 10 — Partitioning Strategy

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines when and how database partitioning may be introduced.

Partitioning is not required on day one, but the schema should avoid choices that make future partitioning impossible.

---

## 2. Core Rule

Do not introduce partitioning before there is a measured need.

Start simple.

Add partitioning when data volume, query latency, retention, or maintenance requires it.

---

## 3. Tables That May Need Partitioning

Future partition candidates:

```text
payments
webhook_deliveries
ledger_entries
journal_entries
audit_logs
provider_attempts
notifications
```

---

## 4. Why These Tables

### payments

High write volume and frequent filtering by merchant/date/status.

### webhook_deliveries

Can grow quickly due to retries and failed endpoints.

### ledger_entries

Financial source records grow with every financial event.

### journal_entries

Every financial event creates a journal entry.

### audit_logs

Audit logs grow with operational usage.

### provider_attempts

Every provider call can create an attempt record.

---

## 5. Recommended Partition Key

For most high-volume tables, partitioning by time is the first candidate.

Examples:

```text
created_at monthly
posted_at monthly
occurred_at monthly
```

For ledger:

```text
ledger_entries by created_at or posted period through journal_entries
```

Exact strategy requires PostgreSQL testing.

---

## 6. Tenant-Based Partitioning

Partitioning by merchant_id is not recommended early.

Reason:

- too many merchants may create too many partitions
- uneven merchant sizes can create imbalance
- time-based reporting is usually more common

Merchant_id should be indexed, not necessarily used as partition key.

---

## 7. MVP Strategy

For MVP:

- no partitioning required
- use good indexes
- monitor query performance
- keep timestamps consistent
- avoid hard deletes
- design for archival later

---

## 8. When to Consider Partitioning

Consider partitioning when:

- payments table reaches tens of millions of rows
- ledger_entries becomes slow for date-range queries
- webhook_deliveries grows rapidly
- audit_logs become expensive to query
- retention/archive processes become slow
- vacuum/maintenance becomes painful

---

## 9. Read Replicas Before Partitioning

For reporting pressure, first consider:

- query optimization
- better indexes
- reporting projections
- read replica
- async exports

Partitioning is not always the first answer.

---

## 10. Reporting Projections

Instead of querying raw financial tables for dashboards, use projections:

```text
merchant_daily_payment_stats
merchant_balance_projections
provider_health_snapshots
webhook_delivery_stats
```

These can reduce pressure on transactional tables.

---

## 11. Archival Strategy

Future archival may move old operational records to cheaper storage.

Candidate records:

- old webhook deliveries
- old provider attempts
- old notification logs

Do not archive ledger entries carelessly.

Ledger requires strict financial retention policy.

---

## 12. Critical Rule

Partitioning is a scaling tool, not a design substitute.

A bad schema with partitioning is still a bad schema.
