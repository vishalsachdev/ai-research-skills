# Advanced API Integration Patterns

Advanced techniques for robust and efficient API integration.

## Parallel Requests

### Using ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

def fetch_user(user_id):
    """Fetch single user"""
    response = requests.get(
        f'{BASE_URL}/users/{user_id}',
        headers=headers,
        timeout=10
    )
    return response.json()

# Fetch 100 users in parallel
user_ids = range(1, 101)

with ThreadPoolExecutor(max_workers=10) as executor:
    # Submit all tasks
    futures = {executor.submit(fetch_user, uid): uid for uid in user_ids}
    
    # Collect results as they complete
    users = []
    for future in as_completed(futures):
        user_id = futures[future]
        try:
            user = future.result()
            users.append(user)
        except Exception as e:
            print(f"User {user_id} failed: {e}")

print(f"Fetched {len(users)} users")
```

### Async with HTTPX

```python
import httpx
import asyncio

async def fetch_user_async(client, user_id):
    response = await client.get(f'{BASE_URL}/users/{user_id}')
    response.raise_for_status()
    return response.json()

async def fetch_all_users_async(user_ids):
    async with httpx.AsyncClient(
        headers=headers,
        timeout=10.0,
        limits=httpx.Limits(max_connections=100)
    ) as client:
        tasks = [fetch_user_async(client, uid) for uid in user_ids]
        users = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Separate successes and errors
        successful = [u for u in users if not isinstance(u, Exception)]
        failed = sum(1 for u in users if isinstance(u, Exception))
        
        print(f"Success: {len(successful)}, Failed: {failed}")
        return successful

# Run
user_ids = range(1, 101)
users = asyncio.run(fetch_all_users_async(user_ids))
```

## Batching Requests

### Batch API Pattern

```python
def batch_create_users(users_list, batch_size=100):
    """Create users in batches"""
    results = []
    
    for i in range(0, len(users_list), batch_size):
        batch = users_list[i:i+batch_size]
        
        response = requests.post(
            f'{BASE_URL}/users/batch',
            json={'users': batch},
            headers=headers
        )
        response.raise_for_status()
        
        results.extend(response.json()['created'])
        print(f"Batch {i//batch_size + 1}: Created {len(batch)} users")
    
    return results
```

### Chunked Processing

```python
import pandas as pd

def process_in_chunks(df, chunk_size=1000):
    """Process DataFrame in chunks"""
    results = []
    
    for start in range(0, len(df), chunk_size):
        chunk = df.iloc[start:start+chunk_size]
        
        # Convert chunk to list of dicts
        records = chunk.to_dict('records')
        
        # Send batch
        response = requests.post(
            f'{BASE_URL}/data/bulk',
            json=records,
            headers=headers
        )
        
        if response.status_code == 200:
            results.append(response.json())
            print(f"Processed rows {start} to {start+len(chunk)}")
        else:
            print(f"Batch failed: {response.status_code}")
    
    return results
```

## Smart Pagination

### Auto-detect Pagination Type

```python
def fetch_all_paginated(url, headers):
    """Auto-detect and handle pagination"""
    all_data = []
    
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    data = response.json()
    
    # Extract items
    if isinstance(data, list):
        all_data = data
        return all_data
    
    # Check for different pagination patterns
    if 'results' in data:
        all_data.extend(data['results'])
    elif 'items' in data:
        all_data.extend(data['items'])
    elif 'data' in data:
        all_data.extend(data['data'])
    
    # Cursor-based
    if 'next_cursor' in data and data['next_cursor']:
        next_url = f"{url}?cursor={data['next_cursor']}"
        all_data.extend(fetch_all_paginated(next_url, headers))
    
    # Link-based (RFC 5988)
    elif 'next' in data and data['next']:
        all_data.extend(fetch_all_paginated(data['next'], headers))
    
    # Page-based
    elif 'page' in data and data['page'] < data.get('total_pages', 0):
        next_page = data['page'] + 1
        next_url = f"{url}?page={next_page}"
        all_data.extend(fetch_all_paginated(next_url, headers))
    
    return all_data
```

### Link Header Pagination

```python
import requests
from requests.utils import parse_header_links

def fetch_all_with_link_header(url, headers):
    """Handle pagination via Link header (GitHub-style)"""
    all_data = []
    
    while url:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        
        all_data.extend(response.json())
        
        # Parse Link header
        link_header = response.headers.get('Link')
        if link_header:
            links = parse_header_links(link_header)
            next_link = [l for l in links if l['rel'] == 'next']
            url = next_link[0]['url'] if next_link else None
        else:
            url = None
    
    return all_data
```

## Caching Strategies

### In-Memory Cache with TTL

```python
import time
from functools import wraps

class SimpleCache:
    def __init__(self, ttl=300):
        self.cache = {}
        self.ttl = ttl
    
    def get(self, key):
        if key in self.cache:
            value, timestamp = self.cache[key]
            if time.time() - timestamp < self.ttl:
                return value
        return None
    
    def set(self, key, value):
        self.cache[key] = (value, time.time())

# Global cache instance
cache = SimpleCache(ttl=3600)

def cached_api_call(url):
    """API call with caching"""
    cached = cache.get(url)
    if cached:
        print(f"Cache hit: {url}")
        return cached
    
    print(f"Cache miss: {url}")
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    data = response.json()
    
    cache.set(url, data)
    return data
```

### Requests-Cache Integration

```python
import requests_cache
from datetime import timedelta

# Setup cache
requests_cache.install_cache(
    'api_cache',
    backend='sqlite',
    expire_after=timedelta(hours=1),
    allowable_methods=['GET', 'POST'],
    allowable_codes=[200, 404]  # Cache successful and not-found
)

# Clear old cache
requests_cache.clear()

# Requests are now cached
response = requests.get(url, headers=headers)

# Check if from cache
print(f"From cache: {response.from_cache}")

# Disable cache for specific request
with requests_cache.disabled():
    fresh_response = requests.get(url, headers=headers)
```

## WebSocket APIs

### Using websocket-client

```python
import websocket
import json

def on_message(ws, message):
    data = json.loads(message)
    print(f"Received: {data}")

def on_error(ws, error):
    print(f"Error: {error}")

def on_close(ws, close_status_code, close_msg):
    print("Connection closed")

def on_open(ws):
    # Send subscription message
    ws.send(json.dumps({
        'action': 'subscribe',
        'channel': 'prices'
    }))

# Connect to WebSocket
ws = websocket.WebSocketApp(
    "wss://api.example.com/stream",
    header={'Authorization': f'Bearer {API_KEY}'},
    on_message=on_message,
    on_error=on_error,
    on_close=on_close,
    on_open=on_open
)

ws.run_forever()
```

### Async WebSocket with websockets

```python
import asyncio
import websockets
import json

async def consume_stream():
    uri = "wss://api.example.com/stream"
    
    async with websockets.connect(
        uri,
        extra_headers={'Authorization': f'Bearer {API_KEY}'}
    ) as websocket:
        # Subscribe
        await websocket.send(json.dumps({
            'action': 'subscribe',
            'channel': 'prices'
        }))
        
        # Receive messages
        while True:
            message = await websocket.recv()
            data = json.loads(message)
            print(f"Received: {data}")

# Run
asyncio.run(consume_stream())
```

## File Uploads

### Upload Files

```python
import requests

# Single file upload
with open('document.pdf', 'rb') as f:
    files = {'file': ('document.pdf', f, 'application/pdf')}
    response = requests.post(
        f'{BASE_URL}/upload',
        files=files,
        headers={'Authorization': f'Bearer {API_KEY}'}
    )

# Multiple files
files = [
    ('files', ('file1.txt', open('file1.txt', 'rb'), 'text/plain')),
    ('files', ('file2.txt', open('file2.txt', 'rb'), 'text/plain'))
]

response = requests.post(f'{BASE_URL}/upload', files=files, headers=headers)

# With additional data
files = {'file': open('document.pdf', 'rb')}
data = {'description': 'Important document', 'category': 'reports'}

response = requests.post(
    f'{BASE_URL}/upload',
    files=files,
    data=data,
    headers=headers
)
```

### Multipart Upload (Large Files)

```python
import requests
import os

def upload_large_file(filepath, chunk_size=1024*1024*5):  # 5MB chunks
    """Upload large file in chunks"""
    filesize = os.path.getsize(filepath)
    filename = os.path.basename(filepath)
    
    # Initiate multipart upload
    response = requests.post(
        f'{BASE_URL}/uploads/initiate',
        json={'filename': filename, 'size': filesize},
        headers=headers
    )
    upload_id = response.json()['upload_id']
    
    # Upload chunks
    parts = []
    with open(filepath, 'rb') as f:
        part_number = 1
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            
            response = requests.put(
                f'{BASE_URL}/uploads/{upload_id}/parts/{part_number}',
                data=chunk,
                headers=headers
            )
            
            parts.append({
                'part_number': part_number,
                'etag': response.headers['ETag']
            })
            
            part_number += 1
            print(f"Uploaded part {part_number-1}/{(filesize + chunk_size - 1) // chunk_size}")
    
    # Complete upload
    response = requests.post(
        f'{BASE_URL}/uploads/{upload_id}/complete',
        json={'parts': parts},
        headers=headers
    )
    
    return response.json()
```

## File Downloads

### Download Files

```python
import requests

# Small file
response = requests.get(f'{BASE_URL}/files/123', headers=headers)

with open('downloaded_file.pdf', 'wb') as f:
    f.write(response.content)

# Large file with streaming
response = requests.get(f'{BASE_URL}/files/large', headers=headers, stream=True)

with open('large_file.zip', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)

# With progress bar
from tqdm import tqdm

response = requests.get(f'{BASE_URL}/files/large', headers=headers, stream=True)
total_size = int(response.headers.get('content-length', 0))

with open('file.zip', 'wb') as f:
    with tqdm(total=total_size, unit='B', unit_scale=True) as pbar:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
            pbar.update(len(chunk))
```

## Webhook Integration

### Receiving Webhooks (Flask)

```python
from flask import Flask, request
import hmac
import hashlib

app = Flask(__name__)

WEBHOOK_SECRET = 'your-webhook-secret'

@app.route('/webhook', methods=['POST'])
def webhook():
    # Verify signature
    signature = request.headers.get('X-Webhook-Signature')
    body = request.get_data()
    
    expected_signature = hmac.new(
        WEBHOOK_SECRET.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(signature, expected_signature):
        return 'Invalid signature', 401
    
    # Process webhook
    data = request.json
    print(f"Received webhook: {data}")
    
    # Process data...
    
    return 'OK', 200

if __name__ == '__main__':
    app.run(port=5000)
```

## API Mocking for Testing

### Using responses library

```python
import responses
import requests

@responses.activate
def test_api_call():
    # Mock API response
    responses.add(
        responses.GET,
        'https://api.example.com/users/123',
        json={'id': 123, 'name': 'John Doe'},
        status=200
    )
    
    # Make request (will use mock)
    response = requests.get('https://api.example.com/users/123')
    assert response.json()['name'] == 'John Doe'

# Mock with callback
def request_callback(request):
    # Dynamic response based on request
    user_id = request.url.split('/')[-1]
    return (200, {}, json.dumps({'id': int(user_id), 'name': f'User {user_id}'}))

@responses.activate
def test_dynamic_mock():
    responses.add_callback(
        responses.GET,
        'https://api.example.com/users/',
        callback=request_callback,
        content_type='application/json',
    )
    
    response = requests.get('https://api.example.com/users/456')
    assert response.json()['id'] == 456
```

## GraphQL Advanced

### Variables and Fragments

```python
import requests

# Query with variables
query = """
query GetUserPosts($userId: ID!, $limit: Int) {
    user(id: $userId) {
        id
        name
        posts(limit: $limit) {
            ...PostFragment
        }
    }
}

fragment PostFragment on Post {
    id
    title
    content
    createdAt
    author {
        name
    }
}
"""

variables = {
    'userId': '123',
    'limit': 10
}

response = requests.post(
    'https://api.example.com/graphql',
    json={'query': query, 'variables': variables},
    headers={'Authorization': f'Bearer {API_KEY}'}
)

data = response.json()['data']
```

### GraphQL Mutations

```python
mutation = """
mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
        id
        title
        createdAt
    }
}
"""

variables = {
    'input': {
        'title': 'New Post',
        'content': 'Post content here',
        'authorId': '123'
    }
}

response = requests.post(
    GRAPHQL_URL,
    json={'query': mutation, 'variables': variables},
    headers=headers
)
```

## API Versioning Strategies

### URL Path Versioning

```python
# Different versions
v1_url = 'https://api.example.com/v1/users'
v2_url = 'https://api.example.com/v2/users'

# Version wrapper
class APIClient:
    def __init__(self, version='v2'):
        self.base_url = f'https://api.example.com/{version}'
    
    def get_users(self):
        return requests.get(f'{self.base_url}/users', headers=headers)

# Use specific version
client_v2 = APIClient(version='v2')
users = client_v2.get_users()
```

### Header-Based Versioning

```python
headers_v1 = {
    'Authorization': f'Bearer {API_KEY}',
    'Accept': 'application/vnd.example.v1+json'
}

headers_v2 = {
    'Authorization': f'Bearer {API_KEY}',
    'Accept': 'application/vnd.example.v2+json'
}

# Request with version header
response = requests.get(
    'https://api.example.com/users',
    headers=headers_v2
)
```

## Circuit Breaker Pattern

```python
import time

class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
    
    def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_success(self):
        self.failure_count = 0
        self.state = 'CLOSED'
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = 'OPEN'

# Usage
breaker = CircuitBreaker()

def api_call():
    return requests.get(f'{BASE_URL}/users', headers=headers)

try:
    response = breaker.call(api_call)
except Exception as e:
    print(f"Circuit breaker prevented call: {e}")
```
