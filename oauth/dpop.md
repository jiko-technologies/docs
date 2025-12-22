# DPoP (Demonstrating Proof of Possession)

DPoP binds access tokens to your client so stolen tokens are useless to attackers.

---

## Why DPoP?

Normal OAuth tokens are "bearer" tokens, whoever has the token can use it. If your token leaks through logs, network interception, or a compromised server, an attacker can use it until it expires.

DPoP ties each token to a cryptographic key pair. Your app proves it holds the private key on every request. Steal the token? Doesn't matter - you can't use it without the key.

Think of it like chip + PIN vs. a credit card number. Anyone can use a stolen card number. With chip + PIN, you need the physical card.

Use DPoP for financial transactions, sensitive data access, or any scenario where token theft would be a big problem.

---

## How It Works

1. Generate a key pair for your client
2. On token requests, include a signed DPoP proof JWT in the `DPoP` header
3. The server binds your token to your public key
4. On API requests, include both the token and a fresh DPoP proof
5. The resource server verifies the proof matches the token's bound key

---

## DPoP Proof Structure

The proof is a JWT with your public key in the header:

```json
{
  "typ": "dpop+jwt",
  "alg": "ES256",
  "jwk": {
    "kty": "EC",
    "crv": "P-256",
    "x": "...",
    "y": "..."
  }
}
```

The payload ties the proof to a specific request:

```json
{
  "jti": "unique-id",
  "htm": "GET",
  "htu": "https://api.business.jiko.io/api/v2/pockets/",
  "iat": 1711910400
}
```

- `jti` - unique ID (prevents replay)
- `htm` - HTTP method
- `htu` - URL you're calling
- `iat` - timestamp (10 second leeway)

---

## Token Request

Include the DPoP proof when getting tokens:

```http
POST /api/oauth2/token HTTP/1.1
Host: auth.jiko.io
Content-Type: application/x-www-form-urlencoded
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2In0...

grant_type=authorization_code&
client_id=your-client-id&
client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&
client_assertion=eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9...
```

Response:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI...",
  "token_type": "DPoP",
  "expires_in": 900
}
```

Note `token_type` is `DPoP`, not `Bearer`.

---

## API Requests

Every request needs the token and a fresh proof:

```http
GET /api/v2/pockets/ HTTP/1.1
Host: api.business.jiko.io
Authorization: DPoP eyJhbGciOiJSUzI1NiIsInR5cCI...
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2In0...
```

Generate a new proof for each request with a unique `jti`, correct `htm`/`htu`, and fresh `iat`.

---

## What DPoP Protects Against

| Threat               | Bearer Token        | DPoP Token              |
| -------------------- | ------------------- | ----------------------- |
| Token in logs        | Attacker can use it | Useless without key     |
| Token intercepted    | Attacker can use it | Useless without key     |
| Token leaked via XSS | Attacker can use it | Useless without key     |
| Replay attacks       | Works until expiry  | Blocked by unique `jti` |

---

## References

- [RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449) - DPoP specification
- [OAuth 2.0 Security BCP](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics) - recommends sender-constrained tokens

