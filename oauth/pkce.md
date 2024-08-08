# PKCE (Proof Key for Code Exchange)

PKCE (Proof Key for Code Exchange) is an extension to the OAuth 2.0 Authorization Code Flow that enhances security, particularly for public clients like mobile and single-page applications. PKCE helps prevent authorization code interception attacks by adding an additional layer of security.

## How PKCE Works

1. **Client Generates Code Verifier and Code Challenge**

   - The client generates a random string called the "code verifier."
   - The client creates a "code challenge" by hashing the code verifier with SHA-256 and encoding it using Base64 URL encoding.

2. **Authorization Request**

   - The client initiates the authorization request to the authorization server, including the code challenge and the method used to generate it (usually `S256` for SHA-256) see [authorization code flow](authorization-code-flow.md).

3. **Token Request with Code Verifier**

   - The client exchanges the authorization code for an access token by including the original code verifier in the request.

4. **Authorization Server Validates Code Verifier**
   - The authorization server verifies that the code verifier matches the code challenge provided during the authorization request. If they match, the server issues an access token.

## PKCE Example Using `bash`

### 1. Generate Code Verifier and Code Challenge

Generate a code verifier and code challenge. Here’s an example using `openssl` and `bash`:

```bash
# Generate a code verifier (random string)
CODE_VERIFIER=$(openssl rand -base64 32 | tr -d '=+/')

# Generate a code challenge (SHA-256 hash of the code verifier)
CODE_CHALLENGE=$(echo -n "$code_verifier" | openssl dgst -binary -sha256 | base64 | tr -d '=+/')

# URL encode the code challenge
CODE_CHALLENGE=$(echo -n "$code_challenge" | sed 's/+/%2B/g; s/\//%2F/g; s/=/%3D/g')

echo $CODE_CHALLENGE
```

Next: [Refresh tokens](refresh-tokens.md)
