# Private Key JWT

Private Key JWT lets you authenticate using asymmetric cryptography instead of a shared secret.

---

## Why Private Key JWT?

Traditional OAuth uses a `client_secret` - a password shared between you and the auth server. Problems with this:

- Secrets leak through logs, config files, git history
- If the server is breached, your secret is exposed
- Rotating a compromised secret requires coordination
- The secret gets sent over the network on every request

With Private Key JWT, you keep a private key and share only the public key with Jiko. To authenticate, you sign a JWT with your private key. Jiko verifies it with your public key.

The private key never leaves your infrastructure.

|               | Client Secret      | Private Key JWT      |
| ------------- | ------------------ | -------------------- |
| Transmission  | Sent every request | Never sent           |
| Server breach | Secret exposed     | Your key safe        |
| Rotation      | Coordinated update | Update independently |

---

## How It Works

1. Sign a JWT with your private key
2. Send it as `client_assertion` in token requests
3. Jiko verifies the signature with your public key

---

## JWT Structure

Header:

```json
{
  "alg": "PS256",
  "typ": "JWT"
}
```

Payload:

```json
{
  "iss": "your-client-id",
  "sub": "your-client-id",
  "aud": "https://auth.jiko.io/api/oauth2/token",
  "iat": 1711910400,
  "exp": 1711910700,
  "jti": "unique-token-id"
}
```

- `iss` / `sub` - your client ID
- `aud` - the token endpoint
- `iat` / `exp` - issued/expiry time (keep it under 5 minutes)
- `jti` - unique ID to prevent replay

---

## Token Request

```http
POST /api/oauth2/token HTTP/1.1
Host: auth.jiko.io
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=authorization-code&
client_id=your-client-id&
client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&
client_assertion=eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Generating Keys

### OpenSSL

```bash
# Generate private key
openssl genrsa -out private_key.pem 2048

# Extract public key
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

### Python

```python
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa

private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)

# Save private key
with open("private_key.pem", "wb") as f:
    f.write(private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.NoEncryption()
    ))

# Save public key
with open("public_key.pem", "wb") as f:
    f.write(private_key.public_key().public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo
    ))
```

---

## Registering Your Key

Add your public key in the Settings page of the Jiko authentication portal. You'll need:

1. Your public key (PEM format)
2. Signing algorithm (PS256 or EdDSA recommended)

Keep your private key secure. Don't commit it to git.

---

## Tips

- Use a secrets manager or HSM in production
- Keep JWTs short-lived (under 5 minutes)
- Use unique `jti` values
- Rotate keys periodically - register the new one before removing the old

---

## References

- [RFC 7523](https://datatracker.ietf.org/doc/html/rfc7523) - JWT client authentication
- [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication) - private_key_jwt method
- [OAuth 2.0 Security BCP](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics) - recommends asymmetric auth
