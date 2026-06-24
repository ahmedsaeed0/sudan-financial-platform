# 07 — Risk Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Risk Domain protects the platform from abuse, operational exposure, excessive transaction volume, suspicious behavior, and unsafe merchant activity.

MVP risk engine should be deterministic and configurable.

Do not start with machine learning.

---

## 2. Responsibilities

The Risk Domain owns:

- merchant risk profiles
- risk rules
- risk checks
- transaction limits
- daily volume limits
- velocity limits
- failed attempt limits
- IP rate limits
- risk alerts
- risk decision logs

The Risk Domain does not own:

- merchant approval lifecycle
- payment creation persistence
- settlement posting

It provides allow/block/flag decisions to other domains.

---

## 3. Key Entities

```text
RiskProfile
RiskRule
RiskDecision
RiskAlert
RiskLimit
BlockedIp
MerchantRiskState
```

---

## 4. MVP Risk Rules

Required MVP rules:

```text
max_transaction_amount
max_daily_volume
max_transactions_per_minute
max_failed_attempts
ip_rate_limit
merchant_status_check
api_key_status_check
```

---

## 5. Risk Check Before Payment

Before payment is created:

1. merchant must be APPROVED
2. merchant must not be SUSPENDED
3. API key must be ACTIVE
4. amount must not exceed max transaction amount
5. daily merchant volume must not exceed max daily volume
6. request velocity must not exceed limit
7. failed attempts must not exceed limit
8. IP must not exceed rate limit

---

## 6. Risk Decisions

Possible decisions:

```text
ALLOW
BLOCK
FLAG
REVIEW_REQUIRED
```

MVP Create Payment usually uses:

```text
ALLOW
BLOCK
```

---

## 7. Risk Failure Codes

```text
MERCHANT_NOT_APPROVED
MERCHANT_SUSPENDED
API_KEY_DISABLED
API_KEY_REVOKED
AMOUNT_LIMIT_EXCEEDED
DAILY_VOLUME_EXCEEDED
RATE_LIMIT_EXCEEDED
FAILED_ATTEMPT_LIMIT_EXCEEDED
IP_RATE_LIMIT_EXCEEDED
```

---

## 8. Risk Profile

A merchant should have one active risk profile.

Fields concept:

```text
merchant_id
max_transaction_amount
max_daily_volume
max_transactions_per_minute
max_failed_attempts
ip_rate_limit
is_active
```

---

## 9. Risk Alert

Risk alerts are created for suspicious or blocked behavior.

Fields concept:

```text
merchant_id
payment_id nullable
api_key_id nullable
rule
severity
details
status
created_at
resolved_at
```

Severity:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

Status:

```text
OPEN
REVIEWING
RESOLVED
DISMISSED
```

---

## 10. Merchant Health Score

Future merchant health score may use:

- payment failure rate
- refund rate
- webhook dead rate
- API error rate
- risk alerts
- reconciliation issues
- abnormal volume changes

---

## 11. Provider Health Score

Provider health may use:

- response time
- timeout rate
- success rate
- error rate
- status verification mismatch rate

Provider health belongs partly to Provider Domain but Risk may consume it.

---

## 12. Settlement Risk

Post-MVP, risk may affect settlement:

- delay settlement
- hold settlement
- require manual review
- reduce limits

MVP may not implement settlement holds but should not block future support.

---

## 13. Domain Events

```text
RiskProfileCreated
RiskProfileUpdated
RiskCheckStarted
RiskCheckPassed
RiskCheckFailed
RiskAlertCreated
RiskAlertResolved
MerchantRiskBlocked
ApiKeyRiskBlocked
IpBlocked
```

---

## 14. Commands

```text
CreateRiskProfile
UpdateRiskProfile
RunPaymentRiskCheck
CreateRiskAlert
ResolveRiskAlert
BlockMerchantRisk
BlockApiKeyRisk
BlockIp
```

---

## 15. Database Tables

```text
risk_profiles
risk_alerts
```

Future:

```text
risk_decisions
blocked_ips
merchant_health_scores
provider_health_scores
```

---

## 16. Testing Requirements

Test:

- amount limit
- daily volume limit
- velocity limit
- failed attempt limit
- suspended merchant blocked
- disabled API key blocked
- risk alert creation
- risk profile update audit

---

## 17. Critical Rules

1. Risk rules must be configurable.
2. Merchant-specific rules must not be hardcoded.
3. Risk blocks payment creation before provider call.
4. Risk decisions should be explainable.
5. High-risk configuration changes must be audited.
