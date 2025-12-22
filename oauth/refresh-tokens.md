# Refresh Tokens

Refresh tokens let you get new access tokens without making the user log in again.

---

## How It Works

When you complete the [Authorization Code Flow](authorization-code-flow.md), you get both an access token and a refresh token. Access tokens expire after 15 minutes. When that happens, use the refresh token to get a new pair.

Refresh tokens expire after 90 days. Once expired, the user needs to log in again.

---

## Token Request

```http
POST /api/oauth2/token HTTP/1.1
Host: auth.jiko.io
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&
refresh_token=dGhpcy1yZWZyZXNoLXRva2VuLi4u...&
client_id=your-client-id&
client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&
client_assertion=eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9...
```

Response:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "bmV3LXJlZnJlc2gtdG9rZW4u...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

---

## Tips

- Store refresh tokens securely - they're long-lived and powerful
- Always save the new refresh token from each response (it rotates)
- Refresh proactively before expiry rather than waiting for a 401
- Client authentication (Private Key JWT) is required for refresh requests

---

## References

- [RFC 6749 - Refreshing an Access Token](https://datatracker.ietf.org/doc/html/rfc6749#section-6)
