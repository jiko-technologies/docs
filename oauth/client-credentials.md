# Client Credentials Flow

## Overview

The **Client Credentials Flow** is used for machine-to-machine (M2M) authentication where no user interaction is required. This flow is ideal for backend services, daemons, or automated processes that need to access protected resources on their own behalf rather than on behalf of a user.

---

## When to Use

- Server-to-server API calls
- Background jobs and scheduled tasks
- Microservices communication
- Automated data synchronization

---

## Getting a Client

Client credentials clients must be provisioned manually by Jiko. Contact Jiko support to request a client for machine-to-machine access.

---

## How It Works

1. The client authenticates directly with the authorization server using its credentials.
2. The authorization server validates the client and issues an access token.
3. The client uses the access token to access protected resources.

Unlike the Authorization Code Flow, there is no user involvement or redirect-based authentication.

---

## Token Request

The client sends a request to the token endpoint with:

- `grant_type`: Must be `client_credentials`
- `client_id`: The client identifier
- `client_assertion_type`: Specifies the JWT format
- `client_assertion`: A signed JWT for client authentication (see [Private Key JWT](./private-key-jwt.md))
- `scope`: The requested scopes (optional)

### Request Example

```http
POST /api/oauth2/token HTTP/1.1
Host: auth.jiko.io
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=your-client-id&
client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&
client_assertion=eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9...&
scope=pockets.read transfers.read
```

### Response Example

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

> Note: The client credentials flow does not return a refresh token. When the access token expires, the client must request a new one using the same flow.

---

## Access Token Lifetime

- Access tokens issued via client credentials have a lifespan of `15 minutes`.
- Since there is no refresh token, the client must request a new access token before the current one expires.

---

## Security Considerations

- Store your private key securely. Never expose it in client-side code or version control.
- Use short-lived JWTs for the `client_assertion` (recommended max 5 minutes).
- Rotate your key pair periodically.
- Request only the scopes your application needs.

---

## Example: Python

```python
import os
import time

import jwt
import requests

# Load your private key
private_key = open("private_key.pem", "rb").read()

# Create the client assertion JWT
now = int(time.time())
claims = {
    "iss": os.getenv("OAUTH_CLIENT_ID"),
    "sub": os.getenv("OAUTH_CLIENT_ID"),
    "aud": "https://auth.jiko.io/api/oauth2/token",
    # For sandbox "aud": "https://authentication-portal.sandbox-api.jikoservices.com/api/oauth2/token",
    "iat": now,
    "exp": now + 300,  # 5 minutes
    "jti": os.urandom(16).hex(),
}

client_assertion = jwt.encode(
    claims,
    private_key,
    algorithm="EdDSA",  # set algorithm to match your key
)

# Request the access token
response = requests.post(
    "https://auth.jiko.io/api/oauth2/token",
    # For sandbox https://authentication-portal.sandbox-api.jikoservices.com/api/oauth2/token
    data={
        "grant_type": "client_credentials",
        "client_id": os.getenv("OAUTH_CLIENT_ID"),
        "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
        "client_assertion": client_assertion,
        "scope": "pockets.read",
    },
)

token_data = response.json()
print(token_data)
access_token = token_data["access_token"]

# Use the access token to call an API
api_response = requests.get(
    "https://api.business.jiko.io/api/v2/pockets/",
    # For sandbox https://customer-api.sandbox-api.jikoservices.com/api/v2/pockets/
    headers={"Authorization": f"Bearer {access_token}"},
)
print(api_response.json())

```

---

### 📚 References

[RFC 6749 - Client Credentials Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4) - OAuth 2.0 Client Credentials Grant specification.

Next: [Scopes](scopes.md)
