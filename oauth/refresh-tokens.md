# Refresh Token Flow

## Overview

In OAuth 2.0 and other authentication systems, the refresh token flow allows clients to obtain a new access token using a refresh token when the original access token has expired. This helps maintain a seamless user experience without requiring the user to re-authenticate frequently.

## Steps in Refresh Token Flow

1. **Initial Authentication Request**:

   - The user logs in and the client application obtains an access token and a refresh token from the authorization server see [authorization code flow](authorization-code-flow.md).

2. **Access Token Expiry**:

   - The access token has a limited lifespan of `15 minutes` and will eventually expire.

3. **Refresh Token Request**:

   - When the access token expires, the client application sends a request to the authorization server using the refresh token to obtain a new access token.

4. **Authorization Server Response**:

   - The authorization server validates the refresh token. If valid, it issues a new access token and a new refresh token.

   * The refresh token expires in `1 day` if this token expires a new login flow is needed.

5. **Client Application Updates**:
   - The client application receives the new access token and uses it for subsequent API requests. It also stores the new refresh token.

## Example

### Initial Authentication

1. **User logs in**:

   - The user provides credentials to the authorization server.
   - The server responds with:
     ```json
     {
       "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
       "refresh_token": "dGhpcy1yZWZyZXNoLWpvaG4tZG9l...",
       "expires_in": 900
     }
     ```

2. **Client stores tokens**:
   - The client application stores the `access_token` and `refresh_token`.

### Access Token Expiry

1. **Access token expires**:
   - The client application attempts to make an API request with the expired access token.
   - The server responds with an error, such as `401 Unauthorized`.

### Refresh Token Request

1. **Client requests a new access token**:

   - The client application sends a request to the authorization server using the refresh token:

     ```http
     POST /api/oauth2/token
     Content-Type: application/x-www-form-urlencoded

     grant_type=refresh_token&
     refresh_token=dGhpcy1yZWZyZXNoLWpvaG4tZG9l...
     ```

2. **Authorization server responds**:
   - The server validates the refresh token and responds with:
     ```json
     {
       "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
       "refresh_token": "dGhpcy1yZWZyZXNoLWpvaG4tZG9l...",
       "expires_in": 900
     }
     ```

### Client Application Updates

1. **Client updates tokens**:

   - The client application receives the new `access_token` and stores it.
   - It also updates the `refresh_token`

2. **Continued API requests**:
   - The client application uses the new `access_token` for API requests.

Next: See [Login flow](login-flow.md)
