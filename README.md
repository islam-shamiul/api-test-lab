# Fintech API Testing Lab (Postman)

## Overview

This project simulates a **fintech transaction flow** using Postman and a mock server.

It covers:

* User onboarding (register + login)
* Authentication handling
* Service selection
* Balance check
* Pre-validation (fraud + eligibility)
* Transaction execution
* Mock-based validation (200 / 403 scenarios)

The goal is to demonstrate **real-world QA testing practices**, not just API calls.

---

## Setup

### 1. Import Files

* Import the Postman Collection
* Import the Environment

---

### 2. Configure Environment

Set:

```text
base_url = https://<your-mock-id>.mock.pstmn.io
```


---

## 🚀 API Flow

```text
1. app-status
2. user_check
3. add_user
4. Login
5. get_products
6. select_product
7. check Balance
8. prevalidation
9. transaction
```

---

## Authentication Flow

* Login generates:

  * `auth_key`
  * `auth_expire`
  * `user_balance`

* Protected APIs validate:

```javascript
if (!authKey || Date.now() > expiry) {
    postman.setNextRequest("Login");
}
```

---

## Balance API

```http
GET /balance/{{user_id}}
```

Stores:

```javascript
pm.environment.set("balance", response.balance);
```

---

## Pre-Validation API (CORE LOGIC)

```http
POST /transactions/pre-validate
```

### Purpose:

* Fraud detection
* Balance validation
* Charge calculation
* Generate `pre_trx_id`

---

### Fraud Checks Implemented

```javascript
let amount = Number(pm.environment.get("trx_amount"));
let balance = Number(pm.environment.get("balance"));

// Fraud
if (amount > 5000) {
    throw new Error("FRAUD_DETECTED");
}

// Invalid
if (amount <= 0) {
    throw new Error("INVALID_AMOUNT");
}

// Balance
if (amount > balance) {
    throw new Error("INSUFFICIENT_BALANCE");
}
```

---

### Success Response

```json
{
  "pre_trx_id": "pre_123456",
  "amount": 1000,
  "charge": 20,
  "total_amount": 1020,
  "eligible": true
}
```

---

## Transaction API

```http
POST /transactions/execute
```

---

### Validation Rules

Transaction succeeds only if:

* `pre_trx_id` matches pre-validation
* `total_amount` matches calculated value
* request is not tampered

---

### Request

```json
{
  "user_id": "{{user_id}}",
  "service_id": "{{service_id}}",
  "amount": "{{trx_amount}}",
  "total_amount": "{{total_amount}}",
  "pre_trx_id": "{{pre_trx_id}}"
}
```

---

## Transaction Test Cases (Implemented)

### Success Cases

* Status code is 200
* Transaction ID exists
* Status = SUCCESS
* `pre_trx_id` present
* `total_amount` matches pre-validation

---

### Failure Case (Mock)

If `pre_trx_id` is wrong:

```json
{
  "pre_trx_id": "invalid_999"
}
```

Response:

* Status: **403**

---

## Mock Server Behavior

Mock server uses **example matching**:

| Case    | pre_trx_id | Response |
| ------- | ---------- | -------- |
| Valid   | pre_123456 | 200      |
| Invalid | wrong_id   | 403      |

---

## Key Learnings

* Postman mock server requires **exact path matching**
* Variables (`{{var}}`) do NOT work in examples
* All environment values are stored as **strings**
* Type conversion is required for validation

---

## Known Issues / Improvements

* Duplicate test cases in transaction script
* `auth_expiry` vs `auth_expire` naming inconsistency
* `aut_key` typo in headers (should be `auth_key`)
* Mock server is static (no dynamic validation)

---

## What This Project Demonstrates

* End-to-end API testing flow
* Auth handling with expiry
* Fraud validation logic
* Pre-validation + execution design
* Mock-based testing strategy

---

## Author

Shamiul Islam
