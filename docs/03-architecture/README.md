# Architecture Documentation

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

This directory defines the technical architecture of the Sudan Financial Platform.

The architecture is designed for a financial infrastructure system, not a simple CRUD application.

---

## Documents

| File | Purpose |
|---|---|
| [01-system-architecture.md](./01-system-architecture.md) | High-level system architecture |
| [02-modular-monolith-structure.md](./02-modular-monolith-structure.md) | Laravel modular monolith structure |
| [03-event-flow.md](./03-event-flow.md) | Domain events and side effects |
| [04-outbox-pattern.md](./04-outbox-pattern.md) | Reliable event persistence and dispatch |
| [05-queue-strategy.md](./05-queue-strategy.md) | Queue separation and worker strategy |
| [06-provider-integration-architecture.md](./06-provider-integration-architecture.md) | External payment provider architecture |
| [07-observability.md](./07-observability.md) | Monitoring, logs, metrics, alerts |
| [08-scaling-strategy.md](./08-scaling-strategy.md) | Growth and scaling phases |

---

## Core Architecture Decisions

- Use Laravel Modular Monolith first.
- Use PostgreSQL as the source of truth.
- Use Redis for queues/cache/rate limits, not financial truth.
- Use Provider Abstraction for EBS and future providers.
- Use Double Entry Ledger for financial truth.
- Use Outbox Pattern for critical events.
- Use State Machines for lifecycle entities.
- Use queues for slow/side-effect operations.

---

## Rule

Do not introduce microservices until real operational bottlenecks are measured.
