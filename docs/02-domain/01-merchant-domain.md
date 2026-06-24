# 01 — Merchant Domain

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

The Merchant Domain manages business onboarding, lifecycle, merchant profile, users, verification, payout destinations, and merchant-level configuration.

Merchant is the primary tenant root of the platform.

---

## 2. Responsibilities

The Merchant Domain owns:

- merchant registration
- merchant profile
- merchant lifecycle status
- merchant verification/KYC workflow
- merchant documents
- merchant payout destinations
- merchant business type
- merchant settlement configuration assignment
- merchant fee profile assignment
- merchant risk profile assignment

---

## 3. Key Entities

```text
Merchant
MerchantUser
MerchantDocument
MerchantPayoutAccount
MerchantSettings
MerchantStatus
```

Related but owned by other domains:

```text
ApiKey -> Identity/Developer Platform
FeeProfile -> Fee Domain
RiskProfile -> Risk Domain
Payment -> Payment Domain
Settlement -> Settlement Domain
```

---

## 4. Merchant Lifecycle

Merchant states:

```text
PENDING
EMAIL_VERIFIED
KYC_SUBMITTED
UNDER_REVIEW
APPROVED
REJECTED
SUSPENDED
```

Valid transitions:

```text
PENDING -> EMAIL_VERIFIED
EMAIL_VERIFIED -> KYC_SUBMITTED
KYC_SUBMITTED -> UNDER_REVIEW
UNDER_REVIEW -> APPROVED
UNDER_REVIEW -> REJECTED
APPROVED -> SUSPENDED
SUSPENDED -> APPROVED
```

---

## 5. Business Types

Initial business types:

```text
ECOMMERCE
DELIVERY
ERP
SAAS
EDUCATION
MARKETPLACE
TRAVEL
AGENCY
SERVICES
OTHER
```

---

## 6. Merchant Users

Merchant users belong to one merchant.

Initial roles:

```text
OWNER
ADMIN
DEVELOPER
FINANCE
SUPPORT
VIEWER
```

### Role Expectations

| Role | Capabilities |
|---|---|
| OWNER | full merchant ownership |
| ADMIN | manage merchant operations |
| DEVELOPER | API keys, webhooks, logs |
| FINANCE | payments, refunds, settlements, reports |
| SUPPORT | search and investigate records |
| VIEWER | read-only access |

---

## 7. Merchant Verification

Verification requires collecting and reviewing business information.

Potential requirements:

- business name
- business type
- email
- phone
- website URL
- commercial registration
- owner identity document
- payout account information

Exact KYC requirements must be confirmed before production.

---

## 8. Payout Destinations

A merchant may have multiple payout destinations.

Rules:

- one primary payout destination should exist before settlement
- payout destination must be verified before production settlement
- sensitive values must be masked or protected
- changes should be audited

---

## 9. Merchant Approval Rules

A merchant can be approved only if:

1. required profile data exists
2. required documents are submitted
3. payout destination is provided
4. internal reviewer approves
5. risk configuration exists or default is assigned
6. fee profile exists or default is assigned

---

## 10. Merchant Suspension Rules

Suspended merchant cannot create LIVE payments.

Suspension reasons may include:

- compliance issue
- risk concern
- suspicious activity
- operational decision
- settlement issue

Suspension must be audited.

---

## 11. Domain Events

Important events:

```text
MerchantRegistered
MerchantEmailVerified
MerchantKycSubmitted
MerchantReviewStarted
MerchantApproved
MerchantRejected
MerchantSuspended
MerchantReactivated
MerchantPayoutAccountAdded
MerchantPayoutAccountVerified
MerchantFeeProfileAssigned
MerchantRiskProfileAssigned
```

---

## 12. Commands

Important commands:

```text
RegisterMerchant
VerifyMerchantEmail
SubmitMerchantKyc
StartMerchantReview
ApproveMerchant
RejectMerchant
SuspendMerchant
ReactivateMerchant
AddMerchantPayoutAccount
VerifyMerchantPayoutAccount
AssignMerchantFeeProfile
AssignMerchantRiskProfile
```

---

## 13. Policies

When `MerchantApproved`:

- allow LIVE API key creation
- notify merchant
- audit approval
- enable payment acceptance if required setup exists

When `MerchantSuspended`:

- block LIVE payment creation
- keep existing historical records accessible
- do not delete merchant records
- audit suspension

---

## 14. Database Tables

Main tables:

```text
merchants
merchant_users
merchant_documents
merchant_payout_accounts
```

Related tables:

```text
api_keys
fee_profiles
risk_profiles
payments
settlements
```

---

## 15. API Surface

Potential merchant/admin APIs:

```text
POST /merchant/register
GET /merchant/profile
PATCH /merchant/profile
POST /merchant/documents
POST /admin/merchants/{id}/approve
POST /admin/merchants/{id}/reject
POST /admin/merchants/{id}/suspend
POST /admin/merchants/{id}/reactivate
```

Exact APIs will be defined in API documentation.

---

## 16. Testing Requirements

Test:

- merchant registration
- lifecycle transitions
- invalid transitions
- approval requirements
- suspension blocks LIVE payment
- payout destination verification
- audit events

---

## 17. Critical Rules

1. Merchant is tenant root.
2. Merchant approval controls LIVE payments.
3. Merchant suspension blocks new LIVE payments.
4. Merchant records are not hard deleted.
5. Sensitive merchant data must be protected.
6. Merchant lifecycle changes must be audited.
