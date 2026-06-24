# 07 — Observability

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how the platform should be monitored and operated.

A financial platform must be observable. If something fails, the team must know what happened, where it happened, and what money or merchant was affected.

---

## 2. Observability Goals

The system should make it easy to answer:

- Did the payment succeed?
- Did ledger posting happen?
- Did webhook delivery happen?
- Did the provider timeout?
- Did settlement complete?
- Did reconciliation find a mismatch?
- Which merchant was affected?
- Which request caused the issue?

---

## 3. MVP Observability Stack

MVP should include:

```text
Application error tracking
Laravel Horizon or queue monitoring
Structured application logs
Health checks
Critical internal alerts
Failed job monitoring
```

Possible tools:

```text
Sentry or equivalent
Laravel Horizon
Telegram/internal alerts
Server monitoring
```

---

## 4. Request Tracing

Every request should have a request ID.

Use request ID in:

- API responses
- logs
- error reports
- support investigations

Suggested field:

```text
request_id
```

---

## 5. Payment Observability

Track:

- payment creation count
- payment success rate
- payment failure rate
- payment pending count
- payment expiry count
- provider timeout count
- average create payment latency

Critical alert:

```text
payment success rate drops below threshold
```

---

## 6. Provider Observability

Track by provider:

- latency
- timeout rate
- error rate
- success rate
- unknown status count
- normalized outcome counts

Critical alert:

```text
provider timeout rate spike
provider unavailable
unknown outcome spike
```

---

## 7. Ledger Observability

Track:

- ledger posting success count
- ledger posting failure count
- duplicate posting attempts
- unbalanced posting rejection
- balance projection rebuild failures

Critical alert:

```text
LedgerPostingFailed
```

Ledger failures are high priority.

---

## 8. Webhook Observability

Track:

- delivery success rate
- delivery failure rate
- retry count
- dead webhook count
- average delivery latency

Critical alert:

```text
WebhookMarkedDead
```

Merchant should be able to see failed/dead webhook deliveries.

---

## 9. Settlement Observability

Track:

- pending settlements
- processing settlements
- failed settlements
- settlement retry count
- settlement completion time
- settlement amount by day

Critical alert:

```text
SettlementFailed
SettlementStuckProcessing
```

---

## 10. Reconciliation Observability

Track:

- reconciliation reports processed
- mismatch count
- unresolved incident count
- incident age

Critical alert:

```text
High mismatch rate
Unresolved critical incident
```

---

## 11. Queue Observability

Track:

- queue depth
- failed jobs
- retry count
- processing latency
- stuck jobs

Queues:

```text
default
webhooks
notifications
settlement
reconciliation
provider-status
reports
outbox
```

---

## 12. Logs

Logs should include:

```text
request_id
merchant_id
payment_id
reference
provider_name
job_id
event_type
```

Do not log:

- API secrets
- webhook secrets
- full sensitive payout details
- provider credentials

---

## 13. Health Checks

Health checks should include:

- application health
- database connectivity
- Redis connectivity
- queue worker health
- provider availability where safe

---

## 14. Future Observability

Future tools:

```text
Prometheus
Grafana
OpenTelemetry
Loki or ELK
Status page
On-call alerts
```

---

## 15. Critical Rules

1. Ledger failure must alert immediately.
2. Provider timeout spikes must be visible.
3. Webhook dead deliveries must be visible.
4. Failed settlements must be visible.
5. Logs must not expose secrets.
6. Every support investigation should start from a reference or request_id.
