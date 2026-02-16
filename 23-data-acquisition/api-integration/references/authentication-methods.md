# Authentication Methods Reference

Complete guide to API authentication methods and implementation patterns.

## API Key Authentication

### Header-Based API Key

Most common method for modern APIs.

```python
import requests

API_KEY = 'your-api-key-here'

# Standard Bearer token
headers = {
    'Authorization': f'Bearer {API_KEY}'
}

response = requests.get('https://api.example.com/data', headers=headers)

# Custom header
headers = {
    'X-API-Key': API_KEY,
    'X-API-Secret': 'your-secret'
}

response = requests.get('https://api.example.com/data', headers=headers)
```

### Query Parameter API Key

Older APIs may use query parameters.

```python
import requests

params = {
    'api_key': API_KEY,
    'format': 'json'
}

response = requests.get('https://api.example.com/data', params=params)

# URL will be: https://api.example.com/data?api_key=xxx&format=json
```

**Security Note**: Avoid query parameters as they can be logged in server logs and browser history.

## Basic Authentication

Username and password encoded in Authorization header.

### Using requests

```python
import requests
from requests.auth import HTTPBasicAuth

# Method 1: Using auth parameter
response = requests.get(
    'https://api.example.com/data',
    auth=HTTPBasicAuth('username', 'password')
)

# Method 2: Tuple shorthand
response = requests.get(
    'https://api.example.com/data',
    auth=('username', 'password')
)

# Method 3: Manual header (less recommended)
import base64

credentials = f'username:password'
encoded = base64.b64encode(credentials.encode()).decode()
headers = {'Authorization': f'Basic {encoded}'}

response = requests.get('https://api.example.com/data', headers=headers)
```

### With Session

```python
import requests

session = requests.Session()
session.auth = ('username', 'password')

# All requests use basic auth
response1 = session.get('https://api.example.com/users')
response2 = session.get('https://api.example.com/products')
```

## OAuth 2.0

Complex but secure authorization framework.

### Authorization Code Flow

Used for web applications.

```python
from requests_oauthlib import OAuth2Session
import os

# Configuration
CLIENT_ID = os.getenv('CLIENT_ID')
CLIENT_SECRET = os.getenv('CLIENT_SECRET')
AUTHORIZATION_BASE_URL = 'https://provider.com/oauth/authorize'
TOKEN_URL = 'https://provider.com/oauth/token'
REDIRECT_URI = 'http://localhost:8080/callback'

# Step 1: Redirect user to authorization URL
oauth = OAuth2Session(CLIENT_ID, redirect_uri=REDIRECT_URI, scope=['read', 'write'])
authorization_url, state = oauth.authorization_url(AUTHORIZATION_BASE_URL)

print(f'Please visit this URL to authorize: {authorization_url}')

# Step 2: User authorizes and is redirected back
authorization_response = input('Paste the full redirect URL here: ')

# Step 3: Fetch access token
token = oauth.fetch_token(
    TOKEN_URL,
    authorization_response=authorization_response,
    client_secret=CLIENT_SECRET
)

print(f'Access token: {token["access_token"]}')

# Step 4: Make API requests
response = oauth.get('https://api.example.com/user/profile')
print(response.json())

# Step 5: Save token for later use
import json
with open('token.json', 'w') as f:
    json.dump(token, f)
```

### Client Credentials Flow

For server-to-server authentication.

```python
from requests_oauthlib import OAuth2Session
from oauthlib.oauth2 import BackendApplicationClient

# Create OAuth2 client
client = BackendApplicationClient(client_id=CLIENT_ID)
oauth = OAuth2Session(client=client)

# Fetch token
token = oauth.fetch_token(
    token_url=TOKEN_URL,
    client_id=CLIENT_ID,
    client_secret=CLIENT_SECRET
)

# Make requests
response = oauth.get('https://api.example.com/data')
```

### Token Refresh

```python
from requests_oauthlib import OAuth2Session

# Load existing token
import json
with open('token.json', 'r') as f:
    token = json.load(f)

# Create session with token
oauth = OAuth2Session(CLIENT_ID, token=token)

# Check if token is expired and refresh
if oauth.token.get('expires_at') and oauth.token['expires_at'] < time.time():
    extra = {
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
    }
    token = oauth.refresh_token(TOKEN_URL, **extra)
    
    # Save new token
    with open('token.json', 'w') as f:
        json.dump(token, f)

# Make request with valid token
response = oauth.get('https://api.example.com/data')
```

### Automatic Token Refresh

```python
from requests_oauthlib import OAuth2Session
from oauthlib.oauth2 import TokenExpiredError

def token_saver(token):
    """Callback to save token when refreshed"""
    with open('token.json', 'w') as f:
        json.dump(token, f)

# Create session with auto-refresh
oauth = OAuth2Session(
    CLIENT_ID,
    token=token,
    auto_refresh_url=TOKEN_URL,
    auto_refresh_kwargs={
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
    },
    token_updater=token_saver
)

# Token automatically refreshed when expired
response = oauth.get('https://api.example.com/data')
```

## JWT (JSON Web Tokens)

Self-contained tokens with claims.

### Decoding JWT

```python
import jwt

token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'

# Decode without verification (for inspection only)
decoded = jwt.decode(token, options={"verify_signature": False})
print(decoded)

# Decode with verification
SECRET_KEY = 'your-secret-key'
try:
    decoded = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    print(f"Valid token: {decoded}")
except jwt.ExpiredSignatureError:
    print("Token has expired")
except jwt.InvalidTokenError:
    print("Invalid token")
```

### Creating JWT

```python
import jwt
import datetime

payload = {
    'user_id': 123,
    'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=1),
    'iat': datetime.datetime.utcnow()
}

SECRET_KEY = 'your-secret-key'
token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')

print(f'JWT: {token}')

# Use in API request
headers = {'Authorization': f'Bearer {token}'}
response = requests.get('https://api.example.com/data', headers=headers)
```

### JWT with RSA Keys

```python
import jwt

# Load private key
with open('private_key.pem', 'r') as f:
    private_key = f.read()

# Create token
payload = {'user_id': 123}
token = jwt.encode(payload, private_key, algorithm='RS256')

# Load public key
with open('public_key.pem', 'r') as f:
    public_key = f.read()

# Verify token
decoded = jwt.decode(token, public_key, algorithms=['RS256'])
```

## AWS Signature V4

For AWS services.

```python
from botocore.auth import SigV4Auth
from botocore.awsrequest import AWSRequest
import requests

# AWS credentials
access_key = 'your-access-key'
secret_key = 'your-secret-key'
region = 'us-east-1'
service = 'execute-api'

# Create request
url = 'https://api.example.com/data'
request = AWSRequest(method='GET', url=url)

# Sign request
SigV4Auth(credentials, service, region).add_auth(request)

# Send signed request
response = requests.get(url, headers=dict(request.headers))
```

## Digest Authentication

More secure than Basic Auth.

```python
from requests.auth import HTTPDigestAuth
import requests

response = requests.get(
    'https://api.example.com/data',
    auth=HTTPDigestAuth('username', 'password')
)
```

## API Key Rotation

Best practice for security.

```python
import os
import time

class APIKeyManager:
    def __init__(self):
        self.primary_key = os.getenv('API_KEY_PRIMARY')
        self.secondary_key = os.getenv('API_KEY_SECONDARY')
        self.current_key = self.primary_key
    
    def get_key(self):
        return self.current_key
    
    def rotate_key(self):
        """Switch to backup key"""
        if self.current_key == self.primary_key:
            self.current_key = self.secondary_key
        else:
            self.current_key = self.primary_key
        
        print(f"Rotated to {'secondary' if self.current_key == self.secondary_key else 'primary'} key")
    
    def make_request(self, url):
        """Make request with current key and rotate on failure"""
        headers = {'Authorization': f'Bearer {self.get_key()}'}
        
        try:
            response = requests.get(url, headers=headers)
            response.raise_for_status()
            return response
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 401:
                # Key might be revoked, try backup
                self.rotate_key()
                headers = {'Authorization': f'Bearer {self.get_key()}'}
                return requests.get(url, headers=headers)
            raise

# Usage
key_manager = APIKeyManager()
response = key_manager.make_request('https://api.example.com/data')
```

## Two-Factor Authentication (2FA)

APIs requiring 2FA.

```python
import requests
import pyotp  # pip install pyotp

# Setup
SECRET = 'your-2fa-secret'
totp = pyotp.TOTP(SECRET)

# Login with username, password, and 2FA code
login_data = {
    'username': 'user',
    'password': 'pass',
    'otp_code': totp.now()  # Generate current code
}

response = requests.post('https://api.example.com/login', json=login_data)

if response.status_code == 200:
    token = response.json()['token']
    # Use token for subsequent requests
```

## Custom Authentication

Some APIs use proprietary schemes.

```python
import hashlib
import hmac
import time

class CustomAuth:
    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret
    
    def __call__(self, request):
        # Generate timestamp
        timestamp = str(int(time.time()))
        
        # Create signature
        message = f"{request.method}{request.url}{timestamp}"
        signature = hmac.new(
            self.api_secret.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()
        
        # Add headers
        request.headers['X-API-Key'] = self.api_key
        request.headers['X-Timestamp'] = timestamp
        request.headers['X-Signature'] = signature
        
        return request

# Usage
auth = CustomAuth('your-api-key', 'your-api-secret')
response = requests.get('https://api.example.com/data', auth=auth)
```

## Environment Variable Management

Secure credential storage.

```python
import os
from pathlib import Path
from dotenv import load_dotenv

# Load from .env file
env_path = Path('.') / '.env'
load_dotenv(dotenv_path=env_path)

# Get credentials
API_KEY = os.getenv('API_KEY')
API_SECRET = os.getenv('API_SECRET')
CLIENT_ID = os.getenv('OAUTH_CLIENT_ID')

# Validate
if not all([API_KEY, API_SECRET]):
    raise ValueError("Missing required credentials in .env file")

# Use in requests
headers = {
    'Authorization': f'Bearer {API_KEY}',
    'X-Secret': API_SECRET
}
```

**.env file**:
```
API_KEY=your_api_key_here
API_SECRET=your_api_secret_here
OAUTH_CLIENT_ID=your_client_id
OAUTH_CLIENT_SECRET=your_client_secret
```

**Add to .gitignore**:
```
.env
token.json
credentials.json
```

## Testing Authentication

```python
import requests

def test_auth(url, auth_headers):
    """Test if authentication works"""
    try:
        response = requests.get(url, headers=auth_headers, timeout=10)
        
        if response.status_code == 200:
            print("✓ Authentication successful")
            return True
        elif response.status_code == 401:
            print("✗ Authentication failed - Invalid credentials")
            return False
        elif response.status_code == 403:
            print("✗ Authentication failed - Forbidden")
            return False
        else:
            print(f"⚠ Unexpected status: {response.status_code}")
            return False
    except requests.exceptions.RequestException as e:
        print(f"✗ Request failed: {e}")
        return False

# Test different auth methods
auth_methods = {
    'Bearer Token': {'Authorization': f'Bearer {API_KEY}'},
    'API Key Header': {'X-API-Key': API_KEY},
    'Basic Auth': {'Authorization': f'Basic {encoded_creds}'}
}

for method_name, headers in auth_methods.items():
    print(f"\nTesting {method_name}:")
    test_auth('https://api.example.com/test', headers)
```

## Rate Limit Headers

Many APIs include rate limit info in headers.

```python
import requests
import time

def check_rate_limit(response):
    """Extract rate limit info from headers"""
    headers = response.headers
    
    limit = headers.get('X-RateLimit-Limit')
    remaining = headers.get('X-RateLimit-Remaining')
    reset = headers.get('X-RateLimit-Reset')
    
    if all([limit, remaining, reset]):
        print(f"Rate limit: {remaining}/{limit} remaining")
        print(f"Resets at: {time.ctime(int(reset))}")
        
        if int(remaining) < 10:
            print("⚠ Warning: Rate limit almost exceeded")
            
        return {
            'limit': int(limit),
            'remaining': int(remaining),
            'reset': int(reset)
        }
    
    return None

# Usage
response = requests.get('https://api.example.com/data', headers=headers)
rate_info = check_rate_limit(response)

if rate_info and rate_info['remaining'] == 0:
    wait_time = rate_info['reset'] - time.time()
    print(f"Waiting {wait_time:.0f} seconds for rate limit reset...")
    time.sleep(wait_time)
```
