## Overview

Jiko APIs aim to implement [RFC 9421](https://datatracker.ietf.org/doc/html/rfc9421) for HTTP Message Signatures. Currently only a subset of the RFC is supported, listed below.

## How It Works

1. The sender selects **HTTP fields** to include (e.g., `@method`, `@path`, `host`, `date`)
2. A **canonical string** is built from those fields
3. The string is **digitally signed** using a private key
4. The signature and metadata are added to HTTP headers (`Signature-Input` and `Signature`)
5. The receiver uses the **public key** to verify the signature

## Signature Headers

### **Signature-Input**

Describes **what was signed** and how.

```http
Signature-Input: sig1=("@method" "host" "date"); keyid="my-key"; alg="ecdsa-p256-sha256"; created=1680000000
```

### **Signature**

Contains the **actual signature** in base64 format.

```http
Signature: sig1=:Base64(signature bytes):
```

### Example Request

```http
GET /api/v2/pockets/ HTTP/1.1
Host: api.business.jiko.io
Date: Tue, 16 May 2023 14:00:00 GMT
Signature-Input: sig1=("@method" "host" "date"); keyid="key-ecdsa-1"; alg="ecdsa-p256-sha256"
Signature: sig1=:MEUCIQD...fakebase64...qGZQ==:
```

The server validates that:

- The fields haven't changed
- The signature matches using the known public key

## Supported Algorithms

RFC 9421 supports a range of algorithms, currently Jiko supports:

- ECDSA (P-256)
- EdDSA (Ed25519)

## What Can Be Signed?

| Field Type     | Example                          |
| -------------- | -------------------------------- |
| HTTP Method    | `@method`                        |
| Request Target | `@path`, `@query`, `@target-uri` |
| Headers        | `host`, `date`, `authorization`  |

Jiko requires the following fields to be signed:

- `@method`
- `@authority`
- `@target-uri`
- `authorization`
- `content-digest` (when request has a body)

### Content-Digest

When a request includes a request body providing a `Content-Digest` header is required. Only `SHA-256` or `SHA-512` digest algorithms are supported. When signing a request that includes a request body, signing the `Content-Digest` header is mandatory.

### Example request with body

```http
POST /api/v2/pockets/ HTTP/1.1
Host: api.business.jiko.io
Date: Tue, 16 May 2023 14:00:00 GMT
Signature-Input: sig1=("@method" "@authority" "@target-uri" "content-digest" "authorization"); keyid="key-ed25519-1"; alg="ecdsa-p256-sha256"
Signature: sig1=:MEUCIQD...fakebase64...qGZQ==:
Content-Digest: sha-512=:RK/0qy18MlBSVnWgjwz6lZ...fakedigest...EWjP/lF5HF9bvEF8FabDg=:
Authorization: ey...faketoken...sJFjajdmJJS3=

{ "pocket_name": "Test pocket" }
```

## Key Management

The protocol requires key identifiers (keyid) to match public keys. You can add your public key in the Settings page of the Jiko authentication portal.
