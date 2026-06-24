# 07 — Admin Security

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines security rules for the internal admin system.

Internal admin users can perform high-risk actions that affect merchants, refunds, settlements, API keys, fees, and risk controls.

---

## 2. Internal Roles

Initial roles:

```text
SUPER_ADMIN
OPERATIONS
FINANCE
RISK
SUPPORT
DEVELOPER
VIEWER
```

---

## 3. Role Responsibilities

| Role | Responsibility |
|---|---|
| SUPER_ADMIN | full internal control |
| OPERATIONS | merchant operations, refunds, settlements |
| FINANCE | reconciliation, settlement review, reports |
| RISK | risk profiles, alerts, suspicious activity |
| SUPPORT | search and investigation |
| DEVELOPER | technical diagnostics, API/webhook support |
| VIEWER | read-only access |

---

## 4. High-Risk Actions

High-risk actions include:

```text
approve merchant
suspend merchant
change fee profile
change risk profile
approve refund
retry refund
create settlement
retry settlement
cancel settlement
rotate API secret
revoke API key
manual ledger adjustment
resolve reconciliation incident
```

---

## 5. Authorization Rules

Every admin action must check:

1. authenticated internal user
2. role permission
3. target resource exists
4. action is allowed by state machine
5. audit event is recorded

---

## 6. Maker-Checker Future

Future high-risk actions should support maker-checker workflow.

Examples:

- fee profile changes
- settlement approval
- large refund approval
- manual ledger adjustment

MVP may use simpler RBAC plus audit, but design should not block maker-checker later.

---

## 7. Admin Session Security

Recommended:

- short session lifetime
- secure cookies
- CSRF protection for web dashboard
- password policy
- login attempt throttling
- future 2FA

---

## 8. Admin Login Alerts

Create security events for:

```text
ADMIN_LOGIN
ADMIN_LOGIN_FAILED
ADMIN_LOGIN_FROM_NEW_IP
```

Future:

- alert on suspicious admin login
- 2FA required for high-risk roles

---

## 9. Support Access Rules

Support users should have limited access.

Allowed:

- view payment timeline
- view webhook status
- view merchant summary
- view refund status

Not allowed unless explicitly permitted:

- approve refunds
- retry settlements
- change fees
- rotate API keys
- edit risk profile

---

## 10. Finance Access Rules

Finance users may access:

- settlement reports
- reconciliation reports
- ledger reports
- fee reports

Finance high-risk actions must be audited.

---

## 11. Risk Access Rules

Risk users may access:

- risk profiles
- risk alerts
- merchant risk state
- failed attempts
- limits

Risk changes must be audited.

---

## 12. Developer Access Rules

Developer/internal technical role may access diagnostics.

Should not access raw secrets.

Allowed:

- request logs with redaction
- provider attempt metadata
- webhook delivery logs
- queue failure diagnostics

Forbidden:

- viewing API secrets
- viewing provider credentials
- changing merchant money state without authorization

---

## 13. Audit Requirements

All high-risk admin actions must produce audit logs.

Audit must include:

```text
actor
role
action
target
old values
new values
ip address
user agent
timestamp
```

---

## 14. Critical Rules

1. Admin access is separate from merchant access.
2. High-risk actions require explicit permission.
3. High-risk actions must be audited.
4. Support role must not have financial write powers by default.
5. Future 2FA and maker-checker should be supported.
