# Authorization Code Flow

The Authorization Code Flow is a common method used to securely obtain an access token from an authorization server. It is typically used in web applications where you need to access resources on behalf of a user.

## Steps:

1. **User Requests Authorization**

   - The user initiates the process by attempting to log in or authorize an application to access their resources. This usually involves clicking a "Login with" or "Authorize" button in the application.

2. **Redirect to Authorization Server**

   - The application redirects the user to an authorization server. This redirect includes:

     - Client ID
     - Requested scopes (what access the application needs)
     - Redirect URI (where the authorization server should send the user after authorization)
     - State (An optional parameter to maintain state between the request and callback)

```http
POST /api/oauth2/authorize
Content-Type: application/x-www-form-urlencoded

response_type=code&
client_id=your-client-id&
redirect_uri=https://your-app.com/callback&
scope=pockets.read&
state=some-random-state
```

3. **User Authenticates and Authorizes**

   - The user logs in to the authorization server and grants or denies permission for the application to access their data.

4. **Authorization Code Issued**

   - If the user approves, the authorization server redirects the user back to the application’s redirect URI with an authorization code.

```http
HTTP/1.1 302 Found
Location: https://your-app.com/callback?code=authorization-code&state=some-random-state
```

5. **Application Requests Token**

   - The application extracts the authorization code from the URL and sends a request to the authorization server’s token endpoint. This request includes:
     - Authorization code
     - Client ID
     - Client secret (a secret key known only to the application and the authorization server)

```http
POST /api/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=authorization-code&
client_id=your-client-id&
client_secret=your-client-secret
```

6. **Token Issued**

   - The authorization server verifies the authorization code and other details. If everything checks out, it responds with an access token and a refresh token. The access token allows the application to make authorized API requests on behalf of the user.

7. **Access Resources**

   - The application uses the access token to request resources from an API or resource server.

8. **Token Refresh**
   - If a refresh token was provided, it can be used to obtain a new access token when the current one expires, without needing the user to reauthorize.

For detailed information and technical breakdowns of these topics see [oauth.com](https://www.oauth.com/oauth2-servers/access-tokens/authorization-code-request/)

Next: See [PKCE](pkce.md)

```

```
