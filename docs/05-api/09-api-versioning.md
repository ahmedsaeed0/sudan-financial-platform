# 09 — API Versioning

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines API versioning and compatibility rules.

Public APIs must be stable because merchants build systems on top of them.

---

## 2. Base Version

Initial public API version:

```text
/api/v1
```

---

## 3. Versioning Style

Use URL path versioning.

Example:

```text
/api/v1/payments
/api/v2/payments
```

Reason:

- simple
- clear
- easy for merchants
- easy to document

---

## 4. Breaking Changes

Breaking changes require a new API version.

Examples:

- removing a field
- changing field type
- changing status meaning
- changing required fields
- changing authentication behavior
- changing webhook signature format
- changing idempotency semantics

---

## 5. Non-Breaking Changes

Usually safe in same version:

- adding optional response fields
- adding new webhook event type
- adding new error details
- adding optional request field
- adding new enum value if clients are expected to handle unknown values

---

## 6. Webhook Versioning

Webhook payloads should follow API compatibility rules.

Avoid breaking existing webhook consumers.

If breaking change is needed:

- create new webhook version
- allow merchant endpoint to choose version
- provide migration period

---

## 7. Deprecation Policy

Before removing an API version:

1. announce deprecation
2. provide migration guide
3. provide timeline
4. monitor usage
5. support old version during migration period

MVP can define the policy later, but should not make breaking changes silently.

---

## 8. Error Code Stability

Error codes are part of API contract.

Do not rename error codes casually.

Adding new error codes is usually non-breaking if clients handle unknown codes safely.

---

## 9. Status Stability

Payment/refund/settlement statuses are part of API contract.

Changing meaning of a status is breaking.

Adding a new status requires careful documentation.

---

## 10. Documentation

Every version should have:

- endpoint docs
- request examples
- response examples
- error codes
- webhook payloads
- migration notes if needed

---

## 11. Critical Rules

1. Public APIs are versioned from day one.
2. Breaking changes require a new version.
3. Error codes must be stable.
4. Webhook payloads must be backward compatible.
5. Do not silently change status semantics.
