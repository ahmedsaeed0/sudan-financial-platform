# 01 — Security Architecture

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the high-level security architecture for the Sudan Financial Platform.

The platform handles payments, merchant data, payout destinations, provider integrations, API keys, webhooks, refunds, settlements, and audit records.

Security must be designed into every layer.

---

## 2. Security Principles

1. Least privilege by default.
2. No plaintext secrets.
3. Audit sensitive actions.
4. Separate merchant users from internal users.
5. Protect financial state with database constraints and domain rules.
6. Do not expose provider internals unnecessarily.
7. Never log secrets.
8. Assume external integrations can fail or timeout.
9. Defense in depth: edge, app, database, operations.

---

## 3. Security Layers

```text
Cloudflare / Edge
    ↓
Nginx / Web Server
    ↓
Laravel Application Security
    ↓
PostgreSQL Data Integrity
    ↓
Redis Queue/Cache Controls
    ↓
Provider Credential Protection
    ↓
Audit and Monitoring
```

---

## 4. Edge Security

Cloudflare responsibilities:

- HTTPS
- DNS protection
- WAF
- DDoS protection
- bot/rate controls where available

Cloudflare does not replace application authentication or authorization.

---

## 5. Application Security

The application must enforce:

- authentication
- authorization
- RBAC
- request validation
- idempotency
- rate limiting
- audit logging
- signed webhooks
- state machine transitions

---

## 6. Authentication Models

### Merchant API

Uses secret API keys.

```text
Authorization: Bearer <secret_key>
```

### Merchant Dashboard

Uses merchant user authentication.

### Internal Admin

Uses internal user authentication and RBAC.

### Webhooks

Use HMAC signature verification by merchant.

---

## 7. Authorization

Merchant users must be scoped to their merchant.

Internal users must be scoped by role and permission.

High-risk actions require explicit permission.

Examples:

- approve refund
- retry settlement
- suspend merchant
- change fee profile
- rotate API key

---

## 8. Data Protection

Sensitive data includes:

- API secrets
- webhook secrets
- payout destination identifiers
- KYC document references
- provider credentials
- customer phone/email
- audit logs

Rules:

- mask sensitive identifiers
- store secrets as hashes or encrypted values
- restrict access
- avoid logging sensitive data

---

## 9. Financial Data Integrity

Use:

- PostgreSQL constraints
- unique indexes
- state machines
- ledger posting uniqueness
- idempotency table
- audit logs

Financial correctness is not only an application concern. Database must protect critical invariants.

---

## 10. Operational Security

Production operations should include:

- SSH key-only access
- limited server users
- backups
- monitoring
- error tracking
- queue monitoring
- secret rotation
- restricted provider credentials

---

## 11. Security Monitoring

Track:

- failed logins
- failed API authentication
- rate limit events
- API key revocations
- suspicious volume spikes
- refund approval actions
- settlement retry actions
- provider auth failures

---

## 12. Critical Rules

1. No plaintext API secrets.
2. No direct uncontrolled admin actions.
3. No hard delete for financial records.
4. No public exposure of sensitive document files.
5. No logs containing secrets.
6. No LIVE payments for unapproved merchants.
