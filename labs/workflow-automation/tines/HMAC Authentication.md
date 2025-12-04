# HMAC Authentication

## Overview

Implementation of HMAC-SHA256 request signing in Tines using Credentials and local_values for secure API authentication without exposing secrets in events.

## What I Built

A workflow that demonstrates cryptographic request signing used by AWS, Stripe, Shopify, and payment processors.

**Workflow:**

```
Webhook → HTTP Request with HMAC Signature → webhook.site (for verification)
```

**Key components:**

- Credential-backed secret storage
- Local calculations using `local_values`
- Canonical string construction
- HMAC-SHA256 signature generation
- Signed headers for authentication

## Implementation

### Credential Setup

Secret stored as Tines Credential (not in events):

- Type: Text
- Name: `hmac_secret_key`
- Reference: `CREDENTIAL.hmac_secret_key`

### HTTP Request with local_values

All HMAC calculations happen in `local_values` before the request is sent:

**local_values configuration:**

```
timestamp: UNIX_TIMESTAMP()
body_object: OBJECT("user_id", 123, "action", "create")
body_json: TO_JSON(LOCAL.body_object)
string_to_sign: CONCAT("POST\n/api/v1/data\n", LOCAL.timestamp, "\n", LOCAL.body_json)
signature: HMAC_SHA256(LOCAL.string_to_sign, CREDENTIAL.hmac_secret_key)
```

**Headers:**

```
Content-Type: application/json
X-Timestamp: =LOCAL.timestamp
X-Signature: =LOCAL.signature
X-Request-ID: =DEFAULT(webhook.body.request_id, "req-001")
```

**Body:**

```
=LOCAL.body_object
```

### Canonical String Format

The string signed by HMAC-SHA256:

```
POST
/api/v1/data
<unix_timestamp>
{"user_id":123,"action":"create"}
```

Each component separated by newlines. Any change to method, path, timestamp, or body produces a different signature.

## How HMAC Provides Security

**Authenticity**: Only parties with the secret key can produce valid signatures. Server recomputes HMAC using received method, path, timestamp, and body - if it matches `X-Signature`, request is authentic.

**Integrity**: Any tampering with request components changes the signature. Attacker can't modify body or headers without knowing the secret key.

**Replay Protection**: Timestamp included in signature prevents replaying old requests. Server rejects requests with stale timestamps (e.g., >5 minutes old).

## Key Learnings

### 1. local_values for Security

Using `local_values` keeps sensitive calculations (signature generation) out of events. Credentials never appear in event history.

**Advantages:**

- Secret stays in Credential storage
- No intermediate Event Transform actions needed
- Cleaner event logs
- All calculations scoped to single action

### 2. Tines Functions for HMAC

Key functions used:

- `UNIX_TIMESTAMP()` - Current Unix time in seconds
- `OBJECT()` - Construct JSON objects
- `TO_JSON()` - Convert to canonical JSON string
- `CONCAT()` - Build multi-line canonical string
- `HMAC_SHA256(text, key)` - Generate signature
- `DEFAULT()` - Provide fallback values safely

### 3. Canonical String Construction

Order and format matter for signature verification:

- Components must match exactly between client and server
- Newlines are significant separators
- JSON must be deterministic (no extra whitespace)
- Both parties must reconstruct identical string

## Real-World Applications

**AWS API Integration**: AWS Signature Version 4 uses HMAC-based signing for all API requests. Required for S3, Lambda, and other AWS services.

**Payment Processor Webhooks**: Stripe, PayPal, and Square use HMAC to verify webhook authenticity. Prevents attackers from spoofing payment notifications.

**Customer API Troubleshooting**: Customer reports "signature mismatch" errors. Debug by comparing canonical string construction, checking timestamp drift, verifying secret key matches.

## Technologies

- **Tines** - Workflow automation with Credentials and local_values
- **HMAC-SHA256** - Cryptographic signing algorithm
- **Tines Formulas** - UNIX_TIMESTAMP, OBJECT, TO_JSON, CONCAT, HMAC_SHA256
- **webhook.site** - Request inspection for testing

---

## Artifacts

### Screenshots

- **workflow-canvas.png** - Complete workflow showing webhook and HTTP request with local_values configuration
- **HMAC in webhook.site.png** - webhook.site showing X-Timestamp, X-Signature, X-Request-ID headers and JSON body

<img width="3002" height="1718" alt="HMAC workflow in tines" src="https://github.com/user-attachments/assets/11af452a-eb47-4bd0-a46d-39f96ec2cef6" />

<img width="2524" height="1332" alt="HMAC in webhook site" src="https://github.com/user-attachments/assets/1fdec22b-bb6f-4a4a-a65f-785064fe9e40" />



---

## Related Labs

- [GitHub Issue Monitor](../workflow-automation/tines/github-issue-monitor/) - Basic Tines formulas and API integration
- [Pagination & Advanced Formulas](../workflow-automation/tines/pagination-formulas/) - Advanced formula patterns
