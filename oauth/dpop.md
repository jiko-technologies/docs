# DPoP (Demonstrating Proof of Possession) Flow

## Overview

**DPoP (Demonstrating Proof of Possession)** is an OAuth 2.0 extension that protects access tokens against misuse by binding them to a specific **client's key**. Unlike traditional Bearer tokens, DPoP ensures that only the client with the corresponding private key can use the token.

---

## How It Works

1. The client generates a **DPoP proof JWT** signed with its private key.
2. The client sends the DPoP proof in the `DPoP` header when:
   - Requesting an access token.
   - Using the access token to access a protected resource.
3. The authorization server issues a **DPoP-bound access token**.
4. The resource server verifies that:
   - The DPoP header is present.
   - The signature is valid.
   - The token was bound to the correct key.

---

## DPoP Proof JWT Structure

### **Header**

The header contains:

- The signing algorithm (e.g., `PS256`, `ES256`, `EdDSA`).
- The token type.

```json
{
  "typ": "dpop+jwt",
  "alg": "ES256",
  "key": "" // Public key for the private key used to sign the DPoP proof
  // in JWK format
}
```

### **Payload**

The should contain the following claims, these claims will be matched against the request being made by the resource server:

- `jti`: A unique identifier.
- `htm`: The HTTP method (e.g., GET, POST).
- `htu`: The HTTP URI of the resource being accessed.
- `iat`: Issued at timestamp. (10 second leeway)

```json
{
  "jti": "unique-id",
  "htm": "GET",
  "htu": "https://api.business.jiko.io/api/v2/pockets/",
  "iat": 1711910400
}
```

### **Signature**

The JWT is signed using a private key using an asymmetric algorithm.

### Token Request with DPoP Proof

When requesting an access token, the client sends:

- `DPoP` header: The signed DPoP proof JWT.
- Standard OAuth parameters (`grant_type`, `client_id`, etc.).

#### Request example

```http
POST /api/oauth2/token HTTP/1.1
Host: auth.jiko.io
Content-Type: application/x-www-form-urlencoded
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2In0.eyJqdGkiOiJ...

grant_type=authorization_code
&client_id=your-client-id
&client_secret=your-client-secret
```

#### Response Example

The server issues a **DPoP-bound access token**:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI...",
  "token_type": "DPoP",
  "expires_in": 3600
}
```

- The `token_type` is `DPoP` instead of `Bearer`.

### Accessing Protected Resources with DPoP

When calling the resource server, the client:

- Includes the `Authorization` header with the DPoP-bound token.
- Sends a new **DPoP proof JWT** in the `DPoP` header.

#### Request example

```http
GET /resource HTTP/1.1
Host: api.business.jiko.io
Authorization: DPoP eyJhbGciOiJSUzI1NiIsInR5cCI...
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2In0.eyJqdGkiOiJ...
```

### 📚 References

[OAuth 2.0 Demonstrating Proof of Possession (DPoP)](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-dpop)

Next: [Refresh tokens](refresh-tokens.md)
