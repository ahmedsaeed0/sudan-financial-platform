# Security Documentation

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## Purpose

This directory defines the security architecture and rules for the Sudan Financial Platform.

Security is part of the product, not an afterthought.

---

## Documents

| File | Purpose |
|---|---|
| [01-security-architecture.md](./01-security-architecture.md) | High-level security architecture |
| [02-api-key-security.md](./02-api-key-security.md) | API key creation, hashing, rotation, revocation |
| [03-webhook-signature-security.md](./03-webhook-signature-security.md) | Webhook HMAC signing and replay protection |
| [04-rate-limiting.md](./04-rate-limiting.md) | API, merchant, IP, and risk rate limits |
| [05-secrets-management.md](./05-secrets-management.md) | Secrets storage and operational handling |
| [06-audit-and-security-events.md](./06-audit-and-security-events.md) | Audit logs and security event model |
| [07-admin-security.md](./07-admin-security.md) | Internal admin security and high-risk actions |

---

## Core Security Rules

1. No plaintext API secrets in database.
2. Webhooks must be signed.
3. Sensitive actions must be audited.
4. Internal admin actions require RBAC.
5. Financial records must not be hard deleted.
6. Secrets must never be logged.
7. Rate limiting must exist at multiple layers.
8. LIVE payment access requires approved merchant.
