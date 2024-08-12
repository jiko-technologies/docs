# FastAPI example

FastAPI example using `starlette` middleware and `authlib`

## 1. Install Dependencies

First, you need to install the necessary packages:

```bash
pip install fastapi[all] httpx python-dotenv authlib uvicorn
```

## 2. Create .env File

```bash
JIKO_CLIENT_ID=your-jiko-client-id
JIKO_CLIENT_SECRET=your-jiko-client-secret
JIKO_METADATA_URL=https://auth.jiko.io/.well-known/openid-configuration
REDIRECT_URI=https://your-callback.com/auth/callback
```

## 3. FastAPI Application Code

```python
import os
from fastapi import FastAPI, Request, Depends, HTTPException
from fastapi.responses import RedirectResponse
from fastapi.security import OAuth2AuthorizationCodeBearer, OAuth2AuthorizationCodeGrant
from authlib.integrations.starlette_client import OAuth
from pydantic import BaseModel
from dotenv import load_dotenv

load_dotenv()

app = FastAPI()

# OAuth2 configuration
oauth = OAuth()
oauth.register(
    name='jiko',
    client_id=os.getenv('JIKO_CLIENT_ID'),
    client_secret=os.getenv('JIKO_CLIENT_SECRET'),
    server_metadata_url=os.getenv('JIKO_METADATA_URL'),
    client_kwargs={'scope': 'openid profile email pockets.read'}
)

class User(BaseModel):
    sub: str
    name: str
    email: str

@app.get("/login")
async def login(request: Request):
    redirect_uri = os.getenv('REDIRECT_URI')
    return await oauth.jiko.authorize_redirect(request, redirect_uri)

@app.route("/auth/callback")
async def auth_callback(request: Request):
    token = await oauth.jiko.authorize_access_token(request)
    user_info = await oauth.jiko.parse_id_token(request, token)

    user = User(
        sub=user_info.get('sub'),
        name=user_info.get('name'),
        email=user_info.get('email')
    )

    # Redirect or respond with user info
    return user

@app.get("/user")
async def read_user(request: Request):
    if 'user' not in request.session:
        raise HTTPException(status_code=401, detail="Not authenticated")
    return request.session['user']

```

## 4. Running the Application

Run the FastAPI application using `uvicorn`:

Explanation:

- **Dependencies**: `authlib` is used for OAuth2 authentication. `httpx` is used for making HTTP requests if needed. `python-dotenv` is used to load environment variables from a `.env `file.

- **OAuth Configuration**: Configure OAuth2 with Jiko by registering it with `authlib`. The `client_kwargs` include the scopes for OAuth2.

- **Login Endpoint**: Redirects users to the Jiko OAuth2 authorization page.

- **Auth Callback Endpoint**: Handles the callback from Jiko, exchanges the authorization code for tokens, retrieves user information, and creates a `User` instance.

- **User Endpoint**: Returns user information if authenticated; otherwise, returns an HTTP 401 Unauthorized error.

Ensure you replace `your-jiko-client-id` and `your-jiko-client-secret `with the actual credentials you get from Jiko. The `REDIRECT_URI` should match the URI you provided during oauth client registration.
