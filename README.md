# api-test-lab

# Fintech API Testing Lab

## Overview

This is a sample project where I tried to simulate a basic fintech API flow using Postman.

The idea was to go beyond simple API calls and include things like:

* authentication handling
* transaction flow
* idempotency (to avoid duplicate transactions)
* basic fraud checks
* retry logic

Everything is mocked using Postman Mock Server.

---

## Setup

### 1. Install Postman

Download and install Postman if you don’t already have it.

---

### 2. Import Collection

* Open Postman
* Click **Import**
* Import:

  * `collection.json`
  * `environment.json`

---

### 3. Setup Environment

Make sure these variables exist:

```
base_url

user_id
username
encrypted_password

auth_key
auth_expiry

user_balance
service_id

trx_amount = 1000

idempotency_key
hmac_signature
```

---

### 4. Setup Mock Server

* Open the collection
* Add **Example responses**
* Click **Mock Collection**
* Copy the mock URL

Set:

```
base_url = https://<your-mock-id>.mock.pstmn.io
```

---

### 5. Run the Flow

Run in order:

```
Health → Register → Login → Services → Balance → Transaction → Statement
```

---

## CI/CD (Automated API Testing)

This project uses **Newman** (Postman CLI runner) to run tests in CI/CD.

---

### Install Newman

```
npm install -g newman
```

---

### Run Tests Locally

```
newman run postman/collection.json \
  -e postman/environment.json
```

---

### GitHub Actions Setup

Create this file:

```
.github/workflows/api-tests.yml
```

---

### CI Workflow

Please check the below file

```
.github/workflows/api-tests.yml
```


### What this does

* Runs tests on every push
* Validates API flow automatically
* Helps catch issues early

---

## Flow

```
Health → Register → Login
       ↓
   Services → Balance
       ↓
   Transaction → Statement
```

---

## Auth

Headers used:

```
Authorization: Bearer {{auth_key}}
X-Auth-Hash: {{encrypted_password}}
```

Transaction only:

```
X-Signature: {{hmac_signature}}
Idempotency-Key: {{idempotency_key}}
```

---

## APIs

* GET /test
* GET /users/99999
* POST /users/add
* POST /auth/login
* GET /products
* GET /users/{{user_id}}
* POST /posts/add
* GET https://httpstat.us/500

---

## Special Logic

### Idempotency

```
if (!pm.environment.get("idempotency_key")) {
    pm.environment.set("idempotency_key", "idem_" + Date.now());
}
```

---

### HMAC

```
let payload = pm.request.body.raw;
pm.environment.set("hmac_signature", btoa(payload + "secret_key"));
```

---

### Auth Expiry

```
if (!authKey || Date.now() > expiry) {
    postman.setNextRequest("Login");
}
```

---

### Retry

```
if (pm.response.code !== 200 && retry < 3) {
    postman.setNextRequest(pm.info.requestName);
}
```

---

## Notes

* Uses mock responses (no real backend)
* Focus is on testing flow and QA practices
* Designed for learning + portfolio

---

## Author

Shamiul Islam
