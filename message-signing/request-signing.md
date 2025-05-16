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
Signature-Input: sig1=("@method" "host" "date"); keyid="my-key"; alg="rsa-pss-sha512"; created=1680000000
```

### **Signature**

Contains the **actual signature** in base64 format.

```http
Signature: sig1=:Base64(signature bytes):
```

## Example Request

```http
GET /api/v2/pockets/ HTTP/1.1
Host: api.business.jiko.io
Date: Tue, 16 May 2023 14:00:00 GMT
Signature-Input: sig1=("@method" "host" "date"); keyid="key-rsa-1"; alg="rsa-v1_5-sha256"
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

Jiko requires signers to sign the following information `@method` `@authority`, `@target-uri` and the `authorization` header, the `content-digest` header is also required when a request includes a body

## Key management

The protocol requires key identifiers (keyid) to match public keys. Key distribution is currently a manual process, with self-serve options coming soon.
