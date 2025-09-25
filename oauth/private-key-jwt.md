# Private Key JWT Client Authentication

## Overview

**Private Key JWT** is a client authentication method used in **OAuth 2.0** and **OpenID Connect**. It allows a client to authenticate itself using a JWT (JSON Web Token) signed with its private key, rather than using a client secret.

## How It Works

1. The client generates a **JWT** and signs it with its **private key**.
2. The client sends the JWT in the `client_assertion` parameter.
3. The server verifies the JWT using the client's **public key**.

---

## JWT Structure

### **Header**

The JWT header specifies:

- The signing algorithm (e.g., `PS256`).
- The token type.

```json
{
  "alg": "PS256",
  "typ": "jwt"
}
```

### **Payload**

The JWT payload contains the following claims:

- `iss`: The client ID.
- `sub`: The client ID (same as `iss`).
- `aud`: The token endpoint of the authorization server.
- `iat`: Issued at timestamp.
- `exp`: Expiration timestamp (max 5 min lifetime recommended).
- `jti`: Unique token identifier to prevent reuse.

```json
{
  "iss": "your-client-id",
  "sub": "your-client-id",
  "aud": "https://auth.jiko.io/api/oauth2/token",
  "iat": 1711910400,
  "exp": 1711911000,
  "jti": "unique-token-id"
}
```

### **Signature**

The client signs the JWT with its private key using an asymmetric algorithm.

Client Authentication Request

When requesting an access token, the client sends:

- `client_assertion_type`: Specifies the JWT format (`urn:ietf:params:oauth:client-assertion-type:jwt-bearer`).
- `client_assertion`: The signed JWT.
- `grant_type`: Authentication flow (`authorization_code`).

#### Request Example

```http
POST /token HTTP/1.1
Host: auth.jiko.io/api/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsInR5cCI...

```

---

## Registering credentials

Contact Jiko support to register your public key with Jiko

### 📚 **References**

[RFC 7523](https://datatracker.ietf.org/doc/html/rfc7523) - JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication.

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication) - Client Authentication using JWT.

Next: [PKCE](pkce.md)
