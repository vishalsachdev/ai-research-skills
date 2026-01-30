---
name: api-integration
description: Provides guidance for integrating with REST APIs using requests and httpx libraries, including authentication methods, rate limiting, pagination handling, error recovery, and async request patterns for efficient data acquisition from web services
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Acquisition, API, REST, HTTP, Requests, HTTPX, Python, Authentication]
dependencies: [requests>=2.31.0, httpx>=0.25.0, requests-oauthlib>=1.3.0]
---

# API Integration

This skill provides expert guidance for integrating with REST APIs using Python's requests and httpx libraries. Learn authentication patterns, pagination handling, rate limiting, error recovery, and async request optimization for efficient data acquisition from web services.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Installation & Setup](#installation--setup)
- [Basic Workflows](#basic-workflows)
- [Advanced Patterns](#advanced-patterns)
- [When to Use vs Alternatives](#when-to-use-vs-alternatives)
- [Common Issues & Solutions](#common-issues--solutions)
- [Best Practices](#best-practices)

## Core Concepts

### HTTP Methods

| Method | Purpose | Has Body | Idempotent |
|--------|---------|----------|-----------|
| GET | Retrieve data | No | Yes |
| POST | Create resource | Yes | No |
| PUT | Update/replace | Yes | Yes |
| PATCH | Partial update | Yes | No |
| DELETE | Remove resource | No | Yes |

### Authentication Types

1. **API Keys**: Simple token in header or query parameter
2. **Bearer Tokens**: OAuth 2.0 tokens in Authorization header
3. **Basic Auth**: Username/password encoded in Authorization header
4. **OAuth 2.0**: Complex flow with access/refresh tokens
5. **JWT**: JSON Web Tokens with claims

### Requests vs HTTPX

| Feature | requests | httpx |
|---------|----------|-------|
| Sync support | ✅ | ✅ |
| Async support | ❌ | ✅ |
| HTTP/2 | ❌ | ✅ |
| Connection pooling | ✅ | ✅ Better |
| Timeout defaults | ⚠️ None | ✅ 5s |
| API | Mature | requests-like |

**Use requests for**: Simple scripts, familiar API, synchronous only
**Use httpx for**: Async operations, HTTP/2, modern features

## Installation & Setup

```bash
# Basic installation
pip install requests

# With HTTPX for async
pip install httpx

# OAuth support
pip install requests-oauthlib

# All together
pip install requests httpx requests-oauthlib python-dotenv
```

## Basic Workflows

### Workflow 1: Simple GET Request with Authentication

**Use Case**: Fetch data from authenticated API endpoint

**Steps**:

1. **Setup API credentials**:
```python
import os
from dotenv import load_dotenv

# Load from .env file
load_dotenv()

API_KEY = os.getenv('API_KEY')
BASE_URL = 'https://api.example.com/v1'
```

2. **Make authenticated request**:
```python
import requests

# Method 1: API key in headers
headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

response = requests.get(
    f'{BASE_URL}/users',
    headers=headers
)

# Check response
if response.status_code == 200:
    data = response.json()
    print(f"Retrieved {len(data)} users")
else:
    print(f"Error: {response.status_code} - {response.text}")
```

3. **Handle response**:
```python
import pandas as pd

# Parse JSON response
data = response.json()

# Convert to DataFrame
if isinstance(data, list):
    df = pd.DataFrame(data)
elif isinstance(data, dict) and 'results' in data:
    df = pd.DataFrame(data['results'])
else:
    df = pd.DataFrame([data])

print(df.head())
```

4. **Error handling**:
```python
try:
    response = requests.get(
        f'{BASE_URL}/users',
        headers=headers,
        timeout=10
    )
    response.raise_for_status()  # Raises HTTPError for bad status
    
    data = response.json()
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
except ValueError:
    print("Invalid JSON response")
```

**Checklist**:
- [ ] Store API keys in environment variables
- [ ] Use appropriate timeout values
- [ ] Check response status code
- [ ] Handle JSON parsing errors
- [ ] Implement retry logic for transient errors

### Workflow 2: Paginated Data Retrieval

**Use Case**: Fetch all data from paginated API

**Steps**:

1. **Offset-based pagination**:
```python
def fetch_all_users_offset(base_url, headers):
    """Fetch all users using offset pagination"""
    all_users = []
    offset = 0
    limit = 100
    
    while True:
        params = {
            'offset': offset,
            'limit': limit
        }
        
        response = requests.get(
            f'{base_url}/users',
            headers=headers,
            params=params,
            timeout=10
        )
        response.raise_for_status()
        
        data = response.json()
        users = data.get('results', [])
        
        if not users:
            break  # No more data
        
        all_users.extend(users)
        offset += limit
        
        print(f"Fetched {len(all_users)} users so far...")
    
    return all_users
```

2. **Page-based pagination**:
```python
def fetch_all_users_page(base_url, headers):
    """Fetch all users using page numbers"""
    all_users = []
    page = 1
    
    while True:
        params = {'page': page, 'per_page': 100}
        
        response = requests.get(
            f'{base_url}/users',
            headers=headers,
            params=params
        )
        response.raise_for_status()
        
        data = response.json()
        users = data.get('results', [])
        
        if not users:
            break
        
        all_users.extend(users)
        
        # Check if more pages exist
        if page >= data.get('total_pages', page):
            break
        
        page += 1
    
    return all_users
```

3. **Cursor-based pagination** (recommended for large datasets):
```python
def fetch_all_users_cursor(base_url, headers):
    """Fetch all users using cursor pagination"""
    all_users = []
    cursor = None
    
    while True:
        params = {'limit': 100}
        if cursor:
            params['cursor'] = cursor
        
        response = requests.get(
            f'{base_url}/users',
            headers=headers,
            params=params
        )
        response.raise_for_status()
        
        data = response.json()
        users = data.get('results', [])
        all_users.extend(users)
        
        # Get next cursor
        cursor = data.get('next_cursor')
        if not cursor:
            break
    
    return all_users
```

4. **Convert to DataFrame**:
```python
import pandas as pd

users = fetch_all_users_cursor(BASE_URL, headers)
df = pd.DataFrame(users)

print(f"Total users: {len(df)}")
df.to_csv('users.csv', index=False)
```

**Checklist**:
- [ ] Identify pagination type (offset, page, cursor)
- [ ] Set appropriate page size (balance API limits and requests)
- [ ] Handle empty results gracefully
- [ ] Implement progress logging
- [ ] Save intermediate results for large datasets

### Workflow 3: POST Request with Data

**Use Case**: Send data to API (create/update resources)

**Steps**:

1. **Prepare data**:
```python
# Create new user
new_user = {
    'name': 'John Doe',
    'email': 'john@example.com',
    'role': 'analyst'
}
```

2. **Send POST request**:
```python
import requests

response = requests.post(
    f'{BASE_URL}/users',
    headers=headers,
    json=new_user  # Automatically sets Content-Type: application/json
)

if response.status_code == 201:  # Created
    created_user = response.json()
    print(f"Created user: {created_user['id']}")
else:
    print(f"Error: {response.status_code} - {response.text}")
```

3. **Bulk create with validation**:
```python
import pandas as pd

# Load users from CSV
df = pd.read_csv('new_users.csv')

created_users = []
failed_users = []

for idx, row in df.iterrows():
    user_data = row.to_dict()
    
    try:
        response = requests.post(
            f'{BASE_URL}/users',
            headers=headers,
            json=user_data,
            timeout=10
        )
        response.raise_for_status()
        
        created_users.append(response.json())
    except requests.exceptions.HTTPError as e:
        failed_users.append({
            'row': idx,
            'data': user_data,
            'error': str(e)
        })
    
    # Rate limiting
    time.sleep(0.1)

print(f"Created: {len(created_users)}, Failed: {len(failed_users)}")

# Save results
if failed_users:
    pd.DataFrame(failed_users).to_csv('failed_users.csv', index=False)
```

4. **Update with PATCH**:
```python
# Partial update
user_id = 123
updates = {
    'role': 'senior_analyst',
    'department': 'Data Science'
}

response = requests.patch(
    f'{BASE_URL}/users/{user_id}',
    headers=headers,
    json=updates
)

if response.status_code == 200:
    updated_user = response.json()
    print(f"Updated user: {updated_user}")
```

**Checklist**:
- [ ] Validate data before sending
- [ ] Use correct content type (JSON vs form data)
- [ ] Check for expected status codes (200, 201, 204)
- [ ] Handle validation errors from API
- [ ] Implement rate limiting for bulk operations

## Advanced Patterns

### Session Management

```python
import requests

# Create session (reuses connection, cookies)
session = requests.Session()

# Set default headers
session.headers.update({
    'Authorization': f'Bearer {API_KEY}',
    'User-Agent': 'MyApp/1.0'
})

# All requests use these headers
response1 = session.get(f'{BASE_URL}/users')
response2 = session.get(f'{BASE_URL}/products')

# Session persists cookies
session.get(f'{BASE_URL}/login')  # Sets cookies
session.get(f'{BASE_URL}/profile')  # Uses cookies automatically
```

### Rate Limiting

```python
import time
from functools import wraps

class RateLimiter:
    def __init__(self, max_calls, period):
        self.max_calls = max_calls
        self.period = period
        self.calls = []
    
    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            
            # Remove old calls
            self.calls = [c for c in self.calls if c > now - self.period]
            
            if len(self.calls) >= self.max_calls:
                sleep_time = self.period - (now - self.calls[0])
                time.sleep(sleep_time)
                self.calls = []
            
            self.calls.append(now)
            return func(*args, **kwargs)
        
        return wrapper

# Usage: 100 calls per minute
@RateLimiter(max_calls=100, period=60)
def api_request(url):
    return requests.get(url, headers=headers)

# Alternative: Use ratelimit library
from ratelimit import limits, sleep_and_retry

@sleep_and_retry
@limits(calls=100, period=60)
def api_request_limited(url):
    return requests.get(url, headers=headers)
```

### Retry Logic with Exponential Backoff

```python
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

def create_session_with_retries():
    """Create session with automatic retry logic"""
    session = requests.Session()
    
    retry_strategy = Retry(
        total=3,                    # Total retries
        backoff_factor=1,           # Wait 1, 2, 4 seconds
        status_forcelist=[429, 500, 502, 503, 504],
        allowed_methods=["GET", "POST", "PUT", "DELETE"]
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    
    return session

# Usage
session = create_session_with_retries()
session.headers.update({'Authorization': f'Bearer {API_KEY}'})

response = session.get(f'{BASE_URL}/users')  # Auto-retries on failure
```

### Async Requests with HTTPX

```python
import httpx
import asyncio

async def fetch_user(client, user_id):
    """Fetch single user asynchronously"""
    response = await client.get(f'{BASE_URL}/users/{user_id}')
    response.raise_for_status()
    return response.json()

async def fetch_all_users_async(user_ids):
    """Fetch multiple users concurrently"""
    async with httpx.AsyncClient(
        headers={'Authorization': f'Bearer {API_KEY}'},
        timeout=10.0
    ) as client:
        tasks = [fetch_user(client, user_id) for user_id in user_ids]
        users = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Filter out errors
        valid_users = [u for u in users if not isinstance(u, Exception)]
        errors = [u for u in users if isinstance(u, Exception)]
        
        print(f"Success: {len(valid_users)}, Errors: {len(errors)}")
        return valid_users

# Run
user_ids = range(1, 101)
users = asyncio.run(fetch_all_users_async(user_ids))
```

### OAuth 2.0 Authentication

```python
from requests_oauthlib import OAuth2Session
import os

# OAuth configuration
CLIENT_ID = os.getenv('OAUTH_CLIENT_ID')
CLIENT_SECRET = os.getenv('OAUTH_CLIENT_SECRET')
AUTHORIZATION_BASE_URL = 'https://api.example.com/oauth/authorize'
TOKEN_URL = 'https://api.example.com/oauth/token'
REDIRECT_URI = 'http://localhost:8000/callback'

# Step 1: Get authorization URL
oauth = OAuth2Session(CLIENT_ID, redirect_uri=REDIRECT_URI)
authorization_url, state = oauth.authorization_url(AUTHORIZATION_BASE_URL)

print(f"Please go to {authorization_url} and authorize access")

# Step 2: Get token from callback
authorization_response = input('Enter the full callback URL: ')

token = oauth.fetch_token(
    TOKEN_URL,
    authorization_response=authorization_response,
    client_secret=CLIENT_SECRET
)

# Step 3: Use token for API requests
response = oauth.get(f'{BASE_URL}/user/profile')
print(response.json())

# Step 4: Refresh token when expired
from requests_oauthlib import OAuth2Session

def refresh_token(token_dict):
    extra = {
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
    }
    
    oauth = OAuth2Session(CLIENT_ID, token=token_dict)
    new_token = oauth.refresh_token(TOKEN_URL, **extra)
    
    return new_token
```

### GraphQL APIs

```python
import requests

def graphql_query(query, variables=None):
    """Execute GraphQL query"""
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    
    data = {'query': query}
    if variables:
        data['variables'] = variables
    
    response = requests.post(
        'https://api.example.com/graphql',
        headers=headers,
        json=data
    )
    response.raise_for_status()
    
    result = response.json()
    
    if 'errors' in result:
        raise Exception(f"GraphQL errors: {result['errors']}")
    
    return result['data']

# Example query
query = """
query GetUsers($limit: Int!) {
    users(limit: $limit) {
        id
        name
        email
        posts {
            title
            createdAt
        }
    }
}
"""

variables = {'limit': 10}
data = graphql_query(query, variables)
users = data['users']
```

## When to Use vs Alternatives

### Use API Integration When:
- ✅ Official API available
- ✅ Real-time or fresh data needed
- ✅ Structured, well-documented endpoints
- ✅ Rate limits are acceptable
- ✅ Authentication supported

### Consider Alternatives When:
- ❌ **Web scraping**: No API available, data only on web pages
- ❌ **Database dumps**: Bulk historical data available
- ❌ **Data warehouse connectors**: Cloud data services (BigQuery, Snowflake)
- ❌ **RSS/Atom feeds**: Blog/news content
- ❌ **Webhooks**: Event-driven real-time updates

### requests vs httpx:

**Use requests when**:
- Simple synchronous scripts
- Familiar with requests API
- Don't need HTTP/2
- Mature ecosystem with many plugins

**Use httpx when**:
- Need async/await support
- Want HTTP/2 support
- Better timeout defaults
- Modern development

## Common Issues & Solutions

### Issue 1: Authentication Failures

**Problem**: 401 Unauthorized or 403 Forbidden

**Solution**: Verify authentication method

```python
# Check which method API uses

# Method 1: Bearer token
headers = {'Authorization': f'Bearer {API_KEY}'}

# Method 2: API key in custom header
headers = {'X-API-Key': API_KEY}

# Method 3: API key in query params
params = {'api_key': API_KEY}
response = requests.get(url, params=params)

# Method 4: Basic auth
from requests.auth import HTTPBasicAuth
response = requests.get(url, auth=HTTPBasicAuth('username', 'password'))

# Debug: Print request headers
import requests
response = requests.get(url, headers=headers)
print(f"Request headers: {response.request.headers}")
```

### Issue 2: Rate Limit Exceeded (429 Error)

**Problem**: Too many requests

**Solution**: Implement retry with backoff

```python
import time

def api_call_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 429:
            # Check Retry-After header
            retry_after = int(response.headers.get('Retry-After', 60))
            print(f"Rate limited. Waiting {retry_after} seconds...")
            time.sleep(retry_after)
            continue
        
        return response
    
    raise Exception("Max retries exceeded")
```

### Issue 3: Timeout Errors

**Problem**: Request hangs or times out

**Solution**: Set appropriate timeouts

```python
# Set both connect and read timeouts
response = requests.get(
    url,
    timeout=(3.05, 27)  # (connect timeout, read timeout)
)

# Or single value for both
response = requests.get(url, timeout=10)

# Increase for slow APIs
response = requests.get(url, timeout=60)
```

### Issue 4: JSON Parsing Errors

**Problem**: `JSONDecodeError` when parsing response

**Solution**: Check content type and handle errors

```python
response = requests.get(url, headers=headers)

# Check content type
content_type = response.headers.get('Content-Type', '')

if 'application/json' in content_type:
    try:
        data = response.json()
    except ValueError:
        print(f"Invalid JSON response: {response.text}")
        data = None
else:
    # Not JSON response
    print(f"Unexpected content type: {content_type}")
    print(f"Response: {response.text}")
```

### Issue 5: SSL Certificate Errors

**Problem**: `SSLError` with certificate verification

**Solution**: Handle certificate issues carefully

```python
# For development only - NEVER in production
response = requests.get(url, verify=False)

# Better: Provide CA bundle
response = requests.get(url, verify='/path/to/ca-bundle.crt')

# Or set environment variable
import os
os.environ['REQUESTS_CA_BUNDLE'] = '/path/to/ca-bundle.crt'
```

### Issue 6: Large Response Bodies

**Problem**: Memory error with large responses

**Solution**: Stream response

```python
# Stream large file
response = requests.get(url, stream=True)

with open('large_file.json', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)

# Or use iter_lines for line-by-line processing
response = requests.get(url, stream=True)
for line in response.iter_lines():
    if line:
        process_line(json.loads(line))
```

## Best Practices

### 1. Use Environment Variables for Secrets

```python
import os
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv('API_KEY')
if not API_KEY:
    raise ValueError("API_KEY not set in environment")
```

### 2. Implement Comprehensive Error Handling

```python
def safe_api_call(url, **kwargs):
    """API call with full error handling"""
    try:
        response = requests.get(url, timeout=10, **kwargs)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.Timeout:
        print("Request timed out")
        return None
    except requests.exceptions.HTTPError as e:
        print(f"HTTP error {e.response.status_code}: {e.response.text}")
        return None
    except requests.exceptions.ConnectionError:
        print("Connection error")
        return None
    except ValueError:
        print("Invalid JSON response")
        return None
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None
```

### 3. Log API Interactions

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def logged_api_call(url, method='GET', **kwargs):
    """API call with logging"""
    logger.info(f"{method} {url}")
    
    response = requests.request(method, url, **kwargs)
    
    logger.info(f"Status: {response.status_code}, Size: {len(response.content)} bytes")
    
    if response.status_code >= 400:
        logger.error(f"Error response: {response.text}")
    
    return response
```

### 4. Cache Responses

```python
import requests_cache

# Enable caching
requests_cache.install_cache(
    'api_cache',
    backend='sqlite',
    expire_after=3600  # 1 hour
)

# Requests are now cached
response = requests.get(url)  # Makes request
response = requests.get(url)  # Uses cache
```

### 5. Validate Responses

```python
from jsonschema import validate, ValidationError

# Define expected schema
user_schema = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"}
    },
    "required": ["id", "name", "email"]
}

# Validate response
response = requests.get(f'{BASE_URL}/users/1')
data = response.json()

try:
    validate(instance=data, schema=user_schema)
    print("Valid response")
except ValidationError as e:
    print(f"Invalid response: {e}")
```

## Additional Resources

- **Authentication Guide**: [references/authentication-methods.md](references/authentication-methods.md)
- **Advanced Patterns**: [references/advanced-patterns.md](references/advanced-patterns.md)
- **Troubleshooting**: [references/troubleshooting-guide.md](references/troubleshooting-guide.md)

## Summary

This skill covered:
- ✅ GET/POST requests with authentication
- ✅ Pagination handling (offset, page, cursor)
- ✅ Rate limiting and retry logic
- ✅ Async requests with httpx
- ✅ OAuth 2.0 authentication
- ✅ Error handling and logging

**Key Takeaways**:
1. Always use environment variables for API keys
2. Implement retry logic with exponential backoff
3. Set appropriate timeouts (connect and read)
4. Handle pagination correctly for complete data
5. Use sessions for multiple requests to same API
6. Cache responses when appropriate
