# 02 — Table Catalog

Status: Draft  
Project: Sudan Financial Platform  
Version: 0.1  
Date: 2026-06-24

---

## 1. Purpose

This document defines the MVP table catalog.

It describes:

- table purpose
- core fields
- ownership domain
- relationships
- important constraints

This is not final migration code.

---

## 2. Identity Tables

### 2.1 internal_users

**Domain:** Identity  
**Purpose:** Platform company users.

Core fields:

```text
id
name
email
phone
password_hash
role
is_active
last_login_at
created_at
updated_at
```

Indexes:

```text
email unique
role
is_active
```

Notes:

- Internal users are separate from merchant users.
- Internal RBAC must be enforced at application layer.

---

### 2.2 merchant_users

**Domain:** Identity / Merchant  
**Purpose:** Users belonging to a merchant.

Core fields:

```text
id
merchant_id
name
email
phone
password_hash
role
is_active
last_login_at
created_at
updated_at
```

Relationships:

```text
merchant_users.merchant_id -> merchants.id
```

Indexes:

```text
merchant_id
email
merchant_id + role
merchant_id + is_active
```

Merchant roles:

```text
OWNER
ADMIN
DEVELOPER
FINANCE
SUPPORT
VIEWER
```

---

## 3. Merchant Tables

### 3.1 merchants

**Domain:** Merchant  
**Purpose:** Merchant business profile and lifecycle.

Core fields:

```text
id
name
business_type
email
phone
website_url
status
approved_at
rejected_at
suspended_at
created_at
updated_at
```

Statuses:

```text
PENDING
EMAIL_VERIFIED
KYC_SUBMITTED
UNDER_REVIEW
APPROVED
REJECTED
SUSPENDED
```

Indexes:

```text
status
email
business_type
created_at
```

Relationships:

```text
merchant has many merchant_users
merchant has many merchant_documents
merchant has many merchant_payout_accounts
merchant has many api_keys
merchant has many payments
merchant has many refunds
merchant has many settlements
merchant has many webhook_endpoints
```

---

### 3.2 merchant_documents

**Domain:** Merchant  
**Purpose:** Merchant verification and onboarding document references.

Core fields:

```text
id
merchant_id
type
file_path
status
verified_at
rejection_reason
created_at
updated_at
```

Types:

```text
COMMERCIAL_REGISTRATION
NATIONAL_ID
BANK_STATEMENT
LOGO
OTHER
```

Statuses:

```text
PENDING
VERIFIED
REJECTED
EXPIRED
```

Indexes:

```text
merchant_id
type
status
```

Notes:

- Files are stored in object storage.
- This table stores references, not binary content.

---

### 3.3 merchant_payout_accounts

**Domain:** Merchant / Settlement  
**Purpose:** Merchant payout destination records.

Core fields:

```text
id
merchant_id
provider_name
account_holder_name
account_reference_masked
account_reference_encrypted
is_primary
status
verified_at
created_at
updated_at
```

Statuses:

```text
PENDING
VERIFIED
REJECTED
DISABLED
```

Indexes:

```text
merchant_id
merchant_id + is_primary
status
```

Rules:

- A merchant should have one primary payout destination.
- Raw payout details must not be exposed in UI.
- Sensitive values must be masked or encrypted.

---

## 4. API Key Tables

### 4.1 api_keys

**Domain:** Identity / Developer Platform  
**Purpose:** Merchant API keys.

Core fields:

```text
id
merchant_id
name
public_key
mode
status
last_used_at
expires_at
created_by
created_at
updated_at
```

Modes:

```text
TEST
LIVE
```

Statuses:

```text
ACTIVE
DISABLED
REVOKED
```

Indexes:

```text
merchant_id
public_key unique
mode
status
```

---

### 4.2 api_key_secrets

**Domain:** Identity / Developer Platform  
**Purpose:** API secret hashes and rotation support.

Core fields:

```text
id
api_key_id
secret_hash
status
active_from
expires_at
revoked_at
created_at
updated_at
```

Statuses:

```text
ACTIVE
EXPIRED
REVOKED
```

Indexes:

```text
api_key_id
api_key_id + status
```

Rules:

- Store secret hash only.
- Never store plaintext secret.
- Rotation may keep two active secrets during grace period.

---

## 5. Fee and Risk Tables

### 5.1 fee_profiles

**Domain:** Fee / Merchant  
**Purpose:** Merchant fee configuration.

Core fields:

```text
id
merchant_id
gateway_percentage
gateway_fixed_fee
provider_percentage
provider_fixed_fee
switch_percentage
switch_fixed_fee
is_active
created_at
updated_at
```

Indexes:

```text
merchant_id
merchant_id + is_active
```

Rules:

- Payment must snapshot calculated fees.
- Old payments must not be recalculated using current fee profile.

---

### 5.2 risk_profiles

**Domain:** Risk  
**Purpose:** Merchant risk limits.

Core fields:

```text
id
merchant_id
max_transaction_amount
max_daily_volume
max_transactions_per_minute
max_failed_attempts
ip_rate_limit
is_active
created_at
updated_at
```

Indexes:

```text
merchant_id
merchant_id + is_active
```

---

## 6. Payment Tables

### 6.1 payments

**Domain:** Payment  
**Purpose:** Payment business records.

Important: this is not the financial source of truth.

Core fields:

```text
id
merchant_id
api_key_id
reference
order_id
amount
currency
gateway_fee
provider_fee
switch_fee
total_fee
net_amount
customer_name
customer_email
customer_phone
description
status
provider_name
provider_reference
failure_reason
callback_url
return_url
metadata_json
expires_at
paid_at
failed_at
cancelled_at
ip_address
user_agent
created_at
updated_at
```

Statuses:

```text
CREATED
PENDING
PROCESSING
SUCCEEDED
FAILED
EXPIRED
CANCELLED
REFUNDED
```

Constraints:

```text
reference unique
merchant_id + order_id unique
```

Indexes:

```text
merchant_id + created_at
merchant_id + status
reference
order_id
provider_reference
customer_phone
```

---

### 6.2 payment_links

**Domain:** Payment  
**Purpose:** Shareable payment links.

Core fields:

```text
id
merchant_id
payment_id nullable
reference
amount
currency
description
status
expires_at
created_at
updated_at
```

Statuses:

```text
ACTIVE
EXPIRED
COMPLETED
CANCELLED
```

---

### 6.3 checkout_sessions

**Domain:** Payment / Checkout  
**Purpose:** Hosted checkout session tracking.

Core fields:

```text
id
payment_id
session_token_hash
status
started_at
completed_at
expires_at
ip_address
user_agent
created_at
updated_at
```

Statuses:

```text
STARTED
COMPLETED
EXPIRED
CANCELLED
```

---

### 6.4 idempotency_keys

**Domain:** Payment / API  
**Purpose:** Prevent duplicate effects from repeated API calls.

Core fields:

```text
id
merchant_id
api_key_id
idempotency_key
endpoint
request_hash
response_json
status
expires_at
created_at
updated_at
```

Constraint:

```text
merchant_id + api_key_id + endpoint + idempotency_key unique
```

Statuses:

```text
PROCESSING
COMPLETED
FAILED
EXPIRED
```

Rules:

- Same key and same request returns same response.
- Same key and different request returns idempotency conflict.

---

## 7. Refund Tables

### 7.1 refunds

**Domain:** Refund  
**Purpose:** Refund request and execution lifecycle.

Core fields:

```text
id
payment_id
merchant_id
amount
currency
reason
requested_by
approved_by
status
provider_reference
failure_reason
requested_at
approved_at
processed_at
created_at
updated_at
```

Statuses:

```text
REQUESTED
PENDING_APPROVAL
APPROVED
REJECTED
PROCESSING
SUCCEEDED
FAILED
```

Indexes:

```text
payment_id
merchant_id
status
created_at
```

Rules:

- MVP supports full refund only.
- Refund approval and refund execution are separate operations.

---

## 8. Webhook Tables

### 8.1 webhook_endpoints

**Domain:** Webhook  
**Purpose:** Merchant webhook endpoint configuration.

Core fields:

```text
id
merchant_id
url
secret_reference
events_json
is_active
created_at
updated_at
```

Indexes:

```text
merchant_id
merchant_id + is_active
```

---

### 8.2 webhook_deliveries

**Domain:** Webhook  
**Purpose:** Delivery tracking and retry state.

Core fields:

```text
id
webhook_endpoint_id
event_type
payload_json
response_code
response_body_truncated
attempts
status
next_retry_at
delivered_at
created_at
updated_at
```

Statuses:

```text
PENDING
DELIVERED
FAILED
RETRYING
DEAD
```

Indexes:

```text
webhook_endpoint_id
status
next_retry_at
event_type
```

---

## 9. Financial Tables

Financial tables are detailed in:

```text
docs/04-database/06-ledger-schema.md
```

Core tables:

```text
ledger_accounts
journal_entries
ledger_entries
balance_projections
```

---

## 10. Settlement Tables

### 10.1 settlements

**Domain:** Settlement  
**Purpose:** Merchant settlement batches.

Core fields:

```text
id
merchant_id
payout_account_id
reference
gross_amount
total_fees
net_amount
transactions_count
status
processed_at
failure_reason
created_at
updated_at
```

Statuses:

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

Indexes:

```text
merchant_id
reference unique
status
processed_at
```

---

## 11. Reconciliation Tables

### 11.1 reconciliation_reports

**Domain:** Reconciliation  
**Purpose:** Uploaded provider/bank reconciliation files.

Core fields:

```text
id
provider_name
file_path
status
uploaded_by
started_at
completed_at
created_at
updated_at
```

### 11.2 reconciliation_incidents

**Domain:** Reconciliation  
**Purpose:** Mismatches detected during reconciliation.

Core fields:

```text
id
reconciliation_report_id
type
severity
internal_reference
external_reference
details_json
status
resolved_by
resolved_at
created_at
updated_at
```

---

## 12. Notification and Audit Tables

### 12.1 notifications

Core fields:

```text
id
recipient_type
recipient_id
type
title
body
status
read_at
created_at
updated_at
```

### 12.2 audit_logs

Core fields:

```text
id
actor_type
actor_id
action
target_type
target_id
old_values_json
new_values_json
ip_address
user_agent
occurred_at
created_at
```

Indexes:

```text
actor_type + actor_id
target_type + target_id
action
occurred_at
```

---

## 13. Review Notes

This catalog should be reviewed before migrations are written.

Questions to confirm:

1. Exact KYC requirements.
2. Exact EBS/provider fields.
3. Exact accounting treatment for provider and switch fees.
4. Whether settlement files are uploaded manually or pulled through integration.
