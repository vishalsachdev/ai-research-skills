# API Integration Troubleshooting Guide

Common problems and solutions when working with REST APIs.

## Connection Issues

### Connection Refused

**Symptom**: `ConnectionRefusedError` or `ConnectionError`

**Causes**:
- Wrong URL or port
- API server down
- Firewall blocking connection
- VPN required

**Solutions**:

```python
import requests
from requests.exceptions import ConnectionError

def test_connection(url):
    """Test API connectivity"""
    try:
        # Try with timeout
        response = requests.get(url, timeout=5)
        print(f"✓ Connected successfully ({response.status_code})")
        return True
    except ConnectionError:
        print("✗ Connection refused - check URL and network")
        return False
    except requests.exceptions.Timeout:
        print("✗ Connection timeout - server may be slow or down")
        return False

# Test different endpoints
endpoints = [
    'https://api.example.com/health',
    'https://api.example.com/v1/status'
]

for endpoint in endpoints:
    print(f"Testing {endpoint}...")
    test_connection(endpoint)
```

### SSL/TLS Errors

**Symptom**: `SSLError: [SSL: CERTIFICATE_VERIFY_FAILED]`

**Solutions**:

```python
import requests
import certifi

# Option 1: Use certifi bundle
response = requests.get(url, verify=certifi.where())

# Option 2: Provide custom CA bundle
response = requests.get(url, verify='/path/to/ca-bundle.crt')

# Option 3: Disable verification (ONLY for development)
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
response = requests.get(url, verify=False)

# Option 4: Update SSL certificates
# pip install --upgrade certifi
```

### DNS Resolution Failures

**Symptom**: `ConnectionError: Failed to resolve hostname`

**Solutions**:

```python
import socket

# Test DNS resolution
try:
    ip = socket.gethostbyname('api.example.com')
    print(f"Resolved to: {ip}")
except socket.gaierror:
    print("DNS resolution failed")
    
    # Try alternative DNS
    import dns.resolver
    resolver = dns.resolver.Resolver()
    resolver.nameservers = ['8.8.8.8']  # Google DNS
    answers = resolver.resolve('api.example.com', 'A')
    print(f"Alternative DNS: {answers[0]}")
```

## Authentication Issues

### 401 Unauthorized

**Symptom**: `401 Unauthorized` response

**Diagnosis**:

```python
import requests

def debug_auth(url, headers):
    """Debug authentication issues"""
    response = requests.get(url, headers=headers)
    
    print(f"Status: {response.status_code}")
    print(f"Headers sent: {response.request.headers}")
    print(f"Response: {response.text}")
    
    if response.status_code == 401:
        print("\nAuthentication failed. Check:")
        print("1. API key is correct and not expired")
        print("2. Authorization header format")
        print("3. API key has required permissions")

debug_auth(url, {'Authorization': f'Bearer {API_KEY}'})
```

**Common fixes**:

```python
# Fix 1: Check header format
# Wrong
headers = {'Authorization': API_KEY}
# Right
headers = {'Authorization': f'Bearer {API_KEY}'}

# Fix 2: Check for extra spaces
api_key = os.getenv('API_KEY').strip()

# Fix 3: URL encode if special characters
from urllib.parse import quote
api_key_encoded = quote(api_key)
```

### Token Expiration

**Symptom**: 401 after some time, but worked initially

**Solution**: Implement token refresh

```python
import time
import json

class TokenManager:
    def __init__(self, token_file='token.json'):
        self.token_file = token_file
        self.token = self.load_token()
    
    def load_token(self):
        try:
            with open(self.token_file, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return None
    
    def save_token(self, token):
        with open(self.token_file, 'w') as f:
            json.dump(token, f)
        self.token = token
    
    def is_expired(self):
        if not self.token:
            return True
        
        expires_at = self.token.get('expires_at', 0)
        # Refresh 5 minutes before expiration
        return time.time() > (expires_at - 300)
    
    def refresh(self):
        """Refresh token logic"""
        response = requests.post(
            'https://api.example.com/refresh',
            json={'refresh_token': self.token['refresh_token']}
        )
        
        new_token = response.json()
        self.save_token(new_token)
        return new_token['access_token']
    
    def get_valid_token(self):
        if self.is_expired():
            return self.refresh()
        return self.token['access_token']

# Usage
token_mgr = TokenManager()
headers = {'Authorization': f'Bearer {token_mgr.get_valid_token()}'}
response = requests.get(url, headers=headers)
```

## Rate Limiting

### 429 Too Many Requests

**Symptom**: `429 Too Many Requests`

**Solution**: Respect rate limits

```python
import time
from functools import wraps

class RateLimitHandler:
    def __init__(self):
        self.retry_count = 0
        self.max_retries = 3
    
    def handle_response(self, response):
        """Handle rate limit response"""
        if response.status_code == 429:
            # Check Retry-After header
            retry_after = response.headers.get('Retry-After')
            
            if retry_after:
                wait_time = int(retry_after)
            else:
                # Exponential backoff
                wait_time = 2 ** self.retry_count
            
            print(f"Rate limited. Waiting {wait_time} seconds...")
            time.sleep(wait_time)
            
            self.retry_count += 1
            return True  # Retry
        
        self.retry_count = 0
        return False  # Don't retry
    
    def make_request(self, url, **kwargs):
        """Make request with rate limit handling"""
        while self.retry_count < self.max_retries:
            response = requests.get(url, **kwargs)
            
            if not self.handle_response(response):
                return response
        
        raise Exception("Max retries exceeded")

# Usage
handler = RateLimitHandler()
response = handler.make_request(url, headers=headers)
```

### Proactive Rate Limiting

```python
import time
from collections import deque

class RateLimiter:
    def __init__(self, max_calls, period):
        """
        max_calls: Maximum number of calls
        period: Time period in seconds
        """
        self.max_calls = max_calls
        self.period = period
        self.calls = deque()
    
    def wait_if_needed(self):
        """Wait if rate limit would be exceeded"""
        now = time.time()
        
        # Remove old calls outside the period
        while self.calls and self.calls[0] < now - self.period:
            self.calls.popleft()
        
        if len(self.calls) >= self.max_calls:
            sleep_time = self.period - (now - self.calls[0])
            if sleep_time > 0:
                print(f"Rate limit: sleeping {sleep_time:.2f}s")
                time.sleep(sleep_time)
        
        self.calls.append(time.time())
    
    def __call__(self, func):
        """Decorator"""
        def wrapper(*args, **kwargs):
            self.wait_if_needed()
            return func(*args, **kwargs)
        return wrapper

# Usage: 10 calls per minute
limiter = RateLimiter(max_calls=10, period=60)

@limiter
def api_call(url):
    return requests.get(url, headers=headers)

# Multiple calls - automatically rate limited
for i in range(20):
    response = api_call(f'{BASE_URL}/users/{i}')
```

## Response Parsing Issues

### JSON Decode Errors

**Symptom**: `JSONDecodeError: Expecting value`

**Solutions**:

```python
import requests
import json

def safe_json_parse(response):
    """Safely parse JSON with debugging"""
    try:
        return response.json()
    except json.JSONDecodeError as e:
        print(f"JSON decode error: {e}")
        print(f"Content-Type: {response.headers.get('Content-Type')}")
        print(f"Response text: {response.text[:500]}")  # First 500 chars
        
        # Try to diagnose
        if not response.text:
            print("Empty response body")
        elif response.text.startswith('<'):
            print("Response looks like HTML, not JSON")
        
        return None

response = requests.get(url, headers=headers)
data = safe_json_parse(response)
```

### Unexpected Response Structure

**Symptom**: `KeyError` or `None` when accessing expected keys

**Solution**: Validate response structure

```python
def extract_data_safely(response):
    """Extract data with fallback logic"""
    try:
        data = response.json()
    except:
        return []
    
    # Try different possible keys
    if isinstance(data, list):
        return data
    
    for key in ['results', 'data', 'items', 'users']:
        if key in data:
            return data[key]
    
    # Wrap single object in list
    if isinstance(data, dict):
        return [data]
    
    return []

# Usage
response = requests.get(url, headers=headers)
items = extract_data_safely(response)
```

## Pagination Issues

### Infinite Loop in Pagination

**Problem**: Never exits pagination loop

**Solution**: Add safety checks

```python
def safe_paginated_fetch(url, max_pages=100):
    """Fetch with pagination safety"""
    all_data = []
    page = 1
    
    while page <= max_pages:
        params = {'page': page, 'per_page': 100}
        response = requests.get(url, params=params, headers=headers)
        response.raise_for_status()
        
        data = response.json()
        items = data.get('results', [])
        
        if not items:
            print(f"No items on page {page}, stopping")
            break
        
        all_data.extend(items)
        print(f"Page {page}: {len(items)} items (total: {len(all_data)})")
        
        # Check if this is the last page
        if page >= data.get('total_pages', page):
            break
        
        # Safety: stop if we're not getting new data
        if len(items) == 0:
            break
        
        page += 1
    
    return all_data
```

### Cursor Pagination Errors

**Problem**: Invalid cursor or cursor expiration

**Solution**: Handle cursor errors

```python
def fetch_with_cursor_retry(url, headers):
    """Cursor pagination with retry"""
    all_data = []
    cursor = None
    retries = 0
    max_retries = 3
    
    while retries < max_retries:
        try:
            params = {'limit': 100}
            if cursor:
                params['cursor'] = cursor
            
            response = requests.get(url, params=params, headers=headers)
            response.raise_for_status()
            
            data = response.json()
            items = data.get('results', [])
            all_data.extend(items)
            
            cursor = data.get('next_cursor')
            if not cursor:
                break
            
            retries = 0  # Reset on success
            
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 400:
                print("Invalid cursor, restarting from beginning")
                cursor = None
                retries += 1
            else:
                raise
    
    return all_data
```

## Timeout Issues

### Read Timeout

**Symptom**: `ReadTimeout` exception

**Solutions**:

```python
import requests

# Increase timeout
response = requests.get(url, headers=headers, timeout=60)

# Separate connect and read timeouts
response = requests.get(
    url,
    headers=headers,
    timeout=(5, 60)  # 5s connect, 60s read
)

# Stream large responses
response = requests.get(url, headers=headers, stream=True, timeout=30)

# Process chunks
for chunk in response.iter_content(chunk_size=8192):
    process(chunk)
```

### Connection Timeout

**Symptom**: `ConnectTimeout` exception

**Solution**: Increase connect timeout and add retry

```python
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

def create_resilient_session():
    session = requests.Session()
    
    retry_strategy = Retry(
        total=3,
        backoff_factor=1,
        status_forcelist=[429, 500, 502, 503, 504],
        allowed_methods=["GET", "POST"]
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    
    return session

session = create_resilient_session()
response = session.get(url, headers=headers, timeout=(10, 30))
```

## Data Type Issues

### Datetime Parsing

**Problem**: API returns dates in various formats

**Solution**: Robust date parsing

```python
from datetime import datetime
import dateutil.parser

def parse_date_flexible(date_string):
    """Parse date from various formats"""
    if not date_string:
        return None
    
    try:
        # Try ISO format first
        return datetime.fromisoformat(date_string.replace('Z', '+00:00'))
    except:
        pass
    
    try:
        # Try dateutil parser (very flexible)
        return dateutil.parser.parse(date_string)
    except:
        pass
    
    # Try common formats
    formats = [
        '%Y-%m-%d',
        '%Y-%m-%dT%H:%M:%S',
        '%Y-%m-%d %H:%M:%S',
        '%m/%d/%Y',
        '%d/%m/%Y'
    ]
    
    for fmt in formats:
        try:
            return datetime.strptime(date_string, fmt)
        except:
            continue
    
    print(f"Could not parse date: {date_string}")
    return None
```

### Number Precision

**Problem**: Decimal/float precision issues

**Solution**: Use Decimal for monetary values

```python
from decimal import Decimal
import json

class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return str(obj)
        return super().default(obj)

# Parse API response with Decimal
response = requests.get(url, headers=headers)
data = json.loads(response.text, parse_float=Decimal)

# Now prices are Decimal instead of float
price = data['price']  # Decimal('19.99')

# Convert back to JSON
json.dumps(data, cls=DecimalEncoder)
```

## Debug Logging

### HTTP Request/Response Logging

```python
import logging
import http.client as http_client

# Enable debug logging
http_client.HTTPConnection.debuglevel = 1

logging.basicConfig()
logging.getLogger().setLevel(logging.DEBUG)
requests_log = logging.getLogger("requests.packages.urllib3")
requests_log.setLevel(logging.DEBUG)
requests_log.propagate = True

# Now all requests are logged with full details
response = requests.get(url, headers=headers)
```

### Custom Request Logger

```python
import logging
import time

class APILogger:
    def __init__(self):
        self.logger = logging.getLogger('api_client')
        self.logger.setLevel(logging.INFO)
        
        handler = logging.FileHandler('api_requests.log')
        formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        )
        handler.setFormatter(formatter)
        self.logger.addHandler(handler)
    
    def log_request(self, method, url, **kwargs):
        """Log API request"""
        self.logger.info(f"{method} {url}")
        if 'headers' in kwargs:
            # Don't log sensitive headers
            safe_headers = {k: v for k, v in kwargs['headers'].items() 
                           if k.lower() not in ['authorization', 'x-api-key']}
            self.logger.debug(f"Headers: {safe_headers}")
    
    def log_response(self, response, duration):
        """Log API response"""
        self.logger.info(
            f"Response: {response.status_code} "
            f"({len(response.content)} bytes in {duration:.2f}s)"
        )
        
        if response.status_code >= 400:
            self.logger.error(f"Error response: {response.text}")

# Usage
logger = APILogger()

def logged_request(method, url, **kwargs):
    logger.log_request(method, url, **kwargs)
    
    start = time.time()
    response = requests.request(method, url, **kwargs)
    duration = time.time() - start
    
    logger.log_response(response, duration)
    
    return response

# Make logged request
response = logged_request('GET', url, headers=headers)
```

## Performance Optimization

### Slow Response Times

**Problem**: API calls taking too long

**Solutions**:

```python
# 1. Use keep-alive (default in Session)
session = requests.Session()
# Reuses connection
for i in range(100):
    session.get(f'{BASE_URL}/users/{i}', headers=headers)

# 2. Compress requests
headers['Accept-Encoding'] = 'gzip, deflate'

# 3. Limit response fields (if API supports)
params = {'fields': 'id,name,email'}  # Only return needed fields
response = session.get(f'{BASE_URL}/users', params=params, headers=headers)

# 4. Use pagination with reasonable page sizes
params = {'per_page': 100}  # Not too small, not too large

# 5. Enable HTTP/2 with httpx
import httpx

async with httpx.AsyncClient(http2=True) as client:
    response = await client.get(url, headers=headers)
```
