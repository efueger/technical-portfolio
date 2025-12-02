# HMAC Authentication Implementation

## Overview

Implemented HMAC-SHA256 request signing from scratch to understand how AWS, Shopify, Stripe, and payment processors authenticate API requests cryptographically.

## What I Built

**Workflow:** Build Components → Create Signature String → Generate HMAC → Send Authenticated Request

**Components:**

1. Signature components (method, path, timestamp, body, secret key)
2. Canonical string creation (standardized format for hashing)
3. HMAC-SHA256 signature generation
4. Authenticated HTTP request with signature in headers

## How HMAC Works

```
Client:
1. Create string: "POST\n/api/data\n1704123456\n{json body}"
2. Hash with secret: HMAC-SHA256(string, secret_key) 
3. Result: a1b2c3d4e5f6... (64-char hex)
4. Send in header: X-Signature: a1b2c3d4...

Server:
1. Receive request
2. Recreate same string from request
3. Hash with same secret
4. Compare signatures
5. Match = authentic ✓ | No match = reject ✗
```

## Why HMAC Over API Keys

**API Keys:** Sent with every request, can be intercepted, static

**HMAC:**

- Secret never transmitted (only signature)
- Signature changes per request (includes timestamp)
- Tamper detection (any change invalidates signature)
- Replay prevention (old timestamps rejected)

## Implementation Details

### Canonical String Format

```
POST
/api/v1/data
1704123456
{"user_id": 123, "action": "create"}
```

Each component on new line. Both client and server must create identical strings.

### Tines Formula

```
=HMAC_SHA256(canonical_string, secret_key)
```

Applies HMAC-SHA256: Takes canonical string, hashes with secret key, returns 64-character hex signature.

**In Event Transform:**

```json
{
  "signature": "=HMAC_SHA256(canonical_string.body, secret_key.body)"
}
```

### Request Headers

```
Content-Type: application/json
X-Timestamp: 1704123456
X-Signature: a1b2c3d4e5f6789...
X-Request-ID: req-001
```

## Key Learnings

### 1. Canonical String Precision

**Problem:** Changed body content without regenerating signature

**Result:** Signature became invalid (would be rejected)

**Lesson:** Any change to method, path, timestamp, or body produces completely different signature. HMAC provides integrity checking.

### 2. Common Pitfalls

- **Whitespace:** `{"key":"value"}` vs `{"key": "value"}` = different signatures
- **Key ordering:** JSON key order must be consistent
- **Line endings:** `\n` vs `\r\n` on different systems
- **Encoding:** URL encoding differences

Both client and server must use exact same serialization.

### 3. Security Model

**What HMAC protects:**

- ✅ Tampering (man-in-the-middle attacks)
- ✅ Replay attacks (with timestamp validation)
- ✅ Forged requests (without secret key)

**What HMAC doesn't protect:**

- ❌ Secret key compromise
- ❌ HTTPS interception (use HTTPS + HMAC together)

## Real-World Applications

### Use Case 1: AWS API Integration

Customer needs to call AWS S3 API programmatically. AWS uses Signature Version 4 (HMAC-SHA256). Every request must be signed with AWS secret key.

**Support approach:** "Let's break down AWS signature process step-by-step. We'll create the canonical request including all required headers..."

### Use Case 2: Webhook Verification

Customer receives Shopify webhooks and wants to verify they're legitimate. Shopify sends `X-Shopify-Hmac-SHA256` header with each webhook.

**Support approach:** "Webhook verification is crucial for security. The HMAC signature proves this came from Shopify. Here's how to verify it..."

### Use Case 3: Payment Processor Integration

Customer keeps getting "Invalid signature" 403 errors from payment API.

**Debugging:**

1. Verify canonical string format matches API docs
2. Confirm secret key is correct
3. Check timestamp format (seconds vs milliseconds)
4. Test with API's example signatures
5. Check for whitespace/encoding issues

## Technologies

- **HMAC-SHA256** - Hash-based Message Authentication Code
- **Tines** - Workflow automation platform
- **RequestBin** - HTTP request inspection
- **Liquid Templates** - Data transformation

## APIs Using HMAC

- AWS API (Signature Version 4)
- Shopify (webhook verification)
- Stripe (webhook signatures)
- Twilio (request validation)
- GitHub (webhook signatures)
- PayPal (IPN verification)

Understanding HMAC means supporting integrations with all these platforms.

---

## Screenshots

*(Add when completing lab)*

- [ ] Tines workflow canvas showing 4-step signature process
- [ ] RequestBin showing authenticated request with X-Signature header
- [ ] Events view showing canonical string and generated signature
