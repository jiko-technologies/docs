# Authorization Code Flow

## Overview

The **Authorization Code Flow** is used to securely obtain an access token from an authorization server on behalf of a user. This flow is typically used in web applications where user authentication and consent are required.

---

## When to Use

- Web applications with server-side backends
- Applications that need to access user data
- Scenarios requiring user authentication and consent

---

## Getting a Client

You can create OAuth clients for the Authorization Code Flow in the Settings page of the Jiko authentication portal.

---

## How It Works

1. The user initiates login by clicking a "Login" or "Authorize" button.
2. The application redirects the user to the authorization server.
3. The user authenticates and grants permission.
4. The authorization server redirects back with an authorization code.
5. The application exchanges the code for an access token.
6. The application uses the access token to access protected resources.

---

## Authorization Request

The application redirects the user to the authorization endpoint with:

- `response_type`: Must be `code`
- `client_id`: The client identifier
- `redirect_uri`: Where to redirect after authorization
- `scope`: The requested scopes
- `state`: A random string to prevent CSRF attacks
- `code_challenge`: PKCE challenge (see [PKCE](./pkce.md))
- `code_challenge_method`: Must be `S256`

### Request Example

```http
GET /api/oauth2/authorize?response_type=code&client_id=your-client-id&redirect_uri=https://your-app.com/callback&scope=pockets.read&state=some-random-state&code_challenge=CODE_CHALLENGE&code_challenge_method=S256
```

### Response Example

After the user authenticates and approves, the authorization server redirects back:

```http
HTTP/1.1 303 See Other
Location: https://your-app.com/callback?code=authorization-code&state=some-random-state
```

---

## Token Request

The application exchanges the authorization code for tokens by sending a request to the token endpoint with:

- `grant_type`: Must be `authorization_code`
- `code`: The authorization code received
- `client_id`: The client identifier
- `client_assertion_type`: Specifies the JWT format
- `client_assertion`: A signed JWT for client authentication (see [Private Key JWT](./private-key-jwt.md))
- `code_verifier`: The PKCE code verifier (see [PKCE](./pkce.md))

### Request Example

```http
POST /api/oauth2/token HTTP/1.1
Host: auth.jiko.io
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=authorization-code&
client_id=your-client-id&
client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&
client_assertion=eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9...&
code_verifier=CODE_VERIFIER
```

### Response Example

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "dGhpcy1yZWZyZXNoLXRva2VuLi4u...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

---

## Access Token Lifetime

- Access tokens have a lifespan of `15 minutes`.
- Refresh tokens have a lifespan of `90 days`.
- Use the refresh token to obtain new access tokens without user re-authentication (see [Refresh Tokens](./refresh-tokens.md)).

---

## Security Considerations

- Always use PKCE to prevent authorization code interception attacks.
- Validate the `state` parameter to prevent CSRF attacks.
- Store tokens securely on the server side.
- Use short-lived JWTs for the `client_assertion` (recommended max 5 minutes).

---

### 📚 References

[RFC 6749 - Authorization Code Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1) - OAuth 2.0 Authorization Code Grant specification.

[oauth.com](https://www.oauth.com/oauth2-servers/access-tokens/authorization-code-request/) - Detailed information and technical breakdowns.

Next: [Client Credentials Flow](./client-credentials.md) | [Private Key JWT](./private-key-jwt.md)
