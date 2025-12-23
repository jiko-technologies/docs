# PKCE (Proof Key for Code Exchange)

PKCE (pronounced "pixy") prevents authorization codes from being stolen and used by attackers.

---

## Why PKCE?

When you use the Authorization Code Flow, the authorization code gets sent back to your app via a redirect URL. The problem: that code can be intercepted.

- On mobile, malicious apps can register the same URL scheme and grab the code
- In browsers, the code ends up in history, logs, or could be captured by browser extensions
- Network proxies or middleware might see the redirect URL

PKCE fixes this by adding a secret that only your app knows. You create a random value (`code_verifier`), hash it (`code_challenge`), and send the hash with your authorization request. When you exchange the code for tokens, you send the original value. The server checks that they match - if someone stole your code, they won't have the verifier.

It's like a coat check: you get a ticket stub, they keep one. To get your coat, both parts need to match.

---

## How It Works

1. Generate a random `code_verifier` (43-128 characters)
2. Hash it with SHA-256 and base64url encode it - that's your `code_challenge`
3. Send `code_challenge` and `code_challenge_method=S256` with your authorization request
4. When exchanging the code, include the original `code_verifier`
5. Server hashes the verifier and checks it matches the challenge

---

## Generating PKCE Values

### bash

```bash
# Generate code verifier
CODE_VERIFIER=$(openssl rand -base64 32 | tr -d '=+/' | cut -c1-43)

# Generate code challenge
CODE_CHALLENGE=$(echo -n "$CODE_VERIFIER" | openssl dgst -binary -sha256 | base64 | tr '+/' '-_' | tr -d '=')

echo "Code Verifier: $CODE_VERIFIER"
echo "Code Challenge: $CODE_CHALLENGE"
```

### Python

```python
import secrets
import hashlib
import base64

code_verifier = secrets.token_urlsafe(32)
code_challenge = base64.urlsafe_b64encode(
    hashlib.sha256(code_verifier.encode()).digest()
).decode().rstrip("=")
```

---

## Requirements

PKCE is **required** for all Authorization Code Flow requests. Only `S256` is supported.

---

## References

- [RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636) - PKCE specification
- [OAuth 2.0 Security BCP](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics) - recommends PKCE for all clients
