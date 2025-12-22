# OAuth

Jiko uses OAuth 2.0 for API authentication. There are two flows depending on your use case:

| Flow | Use Case | Client Setup |
|------|----------|--------------|
| [Authorization Code](authorization-code-flow.md) | User-facing apps that need to act on behalf of users | Self-service via Settings page |
| [Client Credentials](client-credentials.md) | Machine-to-machine, backend services | Contact Jiko support |

## Authentication

All clients authenticate using [Private Key JWT](private-key-jwt.md) - you sign a JWT with your private key, and Jiko verifies it with your public key. No shared secrets.

You can register your public key in the Settings page of the Jiko authentication portal.

## Security Extensions

| Extension | Purpose |
|-----------|---------|
| [PKCE](pkce.md) | Protects authorization codes from interception (required) |
| [DPoP](dpop.md) | Binds tokens to your client so stolen tokens are useless (optional) |

## Token Lifetimes

- Access tokens: 15 minutes
- Refresh tokens: 90 days (Authorization Code Flow only)

See [Refresh Tokens](refresh-tokens.md) for how to get new access tokens without re-authenticating.

## Scopes

See [Scopes](scopes.md) for available permissions.
