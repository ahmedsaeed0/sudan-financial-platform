# 08 — Scaling Strategy

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines how the platform should scale over time.

Scaling should be based on measured bottlenecks, not assumptions.

---

## 2. Core Principle

Start simple, measure, then scale the specific bottleneck.

Do not start with microservices.

---

## 3. Stage 1 — MVP Single Server

Initial setup:

```text
Cloudflare
Nginx
Laravel app
PostgreSQL
Redis
Queue workers
Object storage
```

Good for:

- early development
- pilot merchants
- low to moderate traffic
- validating domain and product

Risks:

- single point of failure
- limited capacity
- app and workers compete for resources

---

## 4. Stage 2 — Separate Database and Redis

Move PostgreSQL and Redis to separate managed/dedicated servers.

Architecture:

```text
App Server
Worker Process
PostgreSQL Server
Redis Server
```

Benefits:

- better resource isolation
- easier database tuning
- better queue stability

---

## 5. Stage 3 — Dedicated Worker Server

Separate web traffic from background jobs.

Architecture:

```text
App Server
Worker Server
PostgreSQL
Redis
```

Good when:

- webhooks increase
- settlement jobs grow
- reconciliation jobs are heavy
- reports are slow

---

## 6. Stage 4 — Multiple App Servers

Use load balancer with multiple app servers.

Architecture:

```text
Load Balancer
├── App Server 1
├── App Server 2
└── App Server N

Worker Server(s)
PostgreSQL
Redis
```

Requirements:

- stateless app servers
- centralized sessions or token auth
- shared storage/object storage
- proper deploy process

---

## 7. Stage 5 — Specialized Workers

Separate workers by queue type:

```text
webhook workers
settlement workers
reconciliation workers
provider-status workers
report workers
```

Benefits:

- critical queues do not starve
- settlement concurrency can be controlled
- webhook throughput can scale independently

---

## 8. Stage 6 — Read Replicas and Reporting Projections

Before microservices, consider:

- PostgreSQL read replicas
- reporting projections
- materialized views where appropriate
- cached merchant dashboard metrics
- async exports

Use for:

- dashboard queries
- reports
- analytics
- admin search

---

## 9. Stage 7 — Selective Service Extraction

Extract services only if needed.

Possible future services:

```text
Webhook Delivery Service
Reporting Service
Risk Service
Settlement Service
Provider Gateway Service
```

Extraction criteria:

- clear module boundary
- independent scaling need
- operational maturity
- reliable event integration
- clear data ownership

---

## 10. Database Scaling

Start with:

- good indexes
- query optimization
- connection pooling if needed
- separate reporting queries

Later:

- read replicas
- partitioning for large tables
- archival strategy
- dedicated analytics store

Partition candidates:

```text
payments
ledger_entries
journal_entries
webhook_deliveries
audit_logs
provider_attempts
```

---

## 11. Queue Scaling

Scale by:

- worker count
- queue separation
- queue priority
- dedicated workers
- concurrency limits

High-risk jobs like settlement need controlled concurrency.

---

## 12. Provider Scaling

Provider scaling depends on external provider limits.

Track:

- provider rate limits
- timeout rate
- latency
- throughput
- provider downtime

Future:

- provider routing
- provider failover
- provider health-based routing

---

## 13. What Not To Do Early

Avoid early:

- microservices
- Kafka/Event streaming without need
- complex Kubernetes setup
- premature sharding
- premature partitioning
- multi-region architecture

These add complexity before product risk is solved.

---

## 14. Scaling Metrics

Monitor:

- API latency
- payment success rate
- provider latency
- queue depth
- failed jobs
- database CPU and locks
- slow queries
- webhook delivery latency
- settlement processing time
- reconciliation processing time

---

## 15. Critical Rules

1. Scale the bottleneck, not the whole system.
2. Do not introduce microservices without measured need.
3. Keep PostgreSQL as financial source of truth.
4. Keep workers idempotent.
5. Keep financial consistency more important than raw throughput.
