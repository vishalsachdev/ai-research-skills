# Anti-Bot Strategies and Advanced Techniques

Guide for handling anti-bot measures and advanced web scraping challenges.

## Common Anti-Bot Measures

### 1. User-Agent Detection

**Problem**: Server blocks requests with default User-Agent

**Solution**: Rotate user agents

```python
import random
import requests

USER_AGENTS = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36',
    'Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X) AppleWebKit/605.1.15',
]

headers = {'User-Agent': random.choice(USER_AGENTS)}
response = requests.get(url, headers=headers)
```

### 2. IP Rate Limiting

**Problem**: Too many requests from same IP

**Solution**: Use proxy rotation

```python
import requests
from itertools import cycle

PROXIES = [
    'http://proxy1.com:8080',
    'http://proxy2.com:8080',
    'http://proxy3.com:8080',
]

proxy_pool = cycle(PROXIES)

def get_with_proxy(url):
    proxy = next(proxy_pool)
    try:
        response = requests.get(
            url,
            proxies={'http': proxy, 'https': proxy},
            timeout=10
        )
        return response
    except requests.exceptions.RequestException:
        # Try next proxy
        return get_with_proxy(url)
```

**Commercial proxy services**:
- Bright Data (formerly Luminati)
- Oxylabs
- ScraperAPI
- Smartproxy

### 3. CAPTCHAs

**Problem**: CAPTCHA challenges

**Solutions**:

1. **Avoid triggering CAPTCHAs**:
```python
# Slow down requests
import time
time.sleep(random.uniform(2, 5))

# Rotate IPs and user agents
# Use residential proxies
# Respect robots.txt
```

2. **Use CAPTCHA solving services** (expensive):
```python
# Example with 2captcha
from twocaptcha import TwoCaptcha

solver = TwoCaptcha('YOUR_API_KEY')
result = solver.recaptcha(
    sitekey='SITE_KEY',
    url='PAGE_URL'
)

# Use result['code'] in your request
```

3. **Headless browser with retry**:
```python
from selenium import webdriver
import time

def solve_captcha_manually():
    """Run browser in non-headless mode for manual solving"""
    options = webdriver.ChromeOptions()
    # Don't use headless mode
    driver = webdriver.Chrome(options=options)
    
    driver.get(url)
    
    # Wait for manual CAPTCHA solving
    input("Solve CAPTCHA and press Enter...")
    
    # Get cookies after solving
    cookies = driver.get_cookies()
    return cookies
```

### 4. JavaScript Challenges

**Problem**: Content loaded via JavaScript

**Solution 1: Selenium**:
```python
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

options = webdriver.ChromeOptions()
options.add_argument('--headless')
driver = webdriver.Chrome(options=options)

driver.get(url)

# Wait for JavaScript to load content
wait = WebDriverWait(driver, 10)
element = wait.until(
    EC.presence_of_element_located((By.CSS_SELECTOR, 'div.dynamic-content'))
)

html = driver.page_source
driver.quit()
```

**Solution 2: requests-html**:
```python
from requests_html import HTMLSession

session = HTMLSession()
response = session.get(url)

# Render JavaScript
response.html.render(timeout=20, sleep=2)

# Now access dynamically loaded content
products = response.html.find('div.product')
```

**Solution 3: Playwright** (faster than Selenium):
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto(url)
    
    # Wait for selector
    page.wait_for_selector('div.product')
    
    # Get content
    content = page.content()
    
    browser.close()
```

### 5. Browser Fingerprinting

**Problem**: Server detects headless browsers

**Solution**: Use stealth plugins

```python
from selenium import webdriver
from selenium_stealth import stealth

options = webdriver.ChromeOptions()
options.add_argument("--headless")
options.add_experimental_option("excludeSwitches", ["enable-automation"])
options.add_experimental_option('useAutomationExtension', False)

driver = webdriver.Chrome(options=options)

# Apply stealth mode
stealth(
    driver,
    languages=["en-US", "en"],
    vendor="Google Inc.",
    platform="Win32",
    webgl_vendor="Intel Inc.",
    renderer="Intel Iris OpenGL Engine",
    fix_hairline=True,
)

driver.get(url)
```

**Playwright stealth**:
```bash
pip install playwright-stealth
```

```python
from playwright.sync_api import sync_playwright
from playwright_stealth import stealth_sync

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    
    # Apply stealth
    stealth_sync(page)
    
    page.goto(url)
```

### 6. Cookie and Session Tracking

**Problem**: Server requires valid session

**Solution**: Manage sessions properly

```python
import requests
import pickle

# Create session
session = requests.Session()

# Set cookies if you have them
session.cookies.update({
    'session_id': 'abc123',
    'user_pref': 'en-US'
})

# Make requests (cookies persist)
response1 = session.get('https://example.com/page1')
response2 = session.get('https://example.com/page2')

# Save session for later
with open('session.pkl', 'wb') as f:
    pickle.dump(session.cookies, f)

# Load session
with open('session.pkl', 'rb') as f:
    session.cookies.update(pickle.load(f))
```

### 7. Honeypot Links

**Problem**: Hidden links that catch bots

**Solution**: Check if elements are visible

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get(url)

links = driver.find_elements(By.TAG_NAME, 'a')

for link in links:
    # Check if link is visible
    if link.is_displayed():
        # Safe to follow
        href = link.get_attribute('href')
    else:
        # Skip hidden links (potential honeypots)
        continue
```

### 8. Request Pattern Detection

**Problem**: Bot-like request patterns

**Solution**: Randomize behavior

```python
import random
import time

def human_like_delay():
    """Random delay mimicking human behavior"""
    return random.uniform(1.5, 4.5)

def scrape_with_human_behavior(urls):
    for i, url in enumerate(urls):
        # Random delay
        time.sleep(human_like_delay())
        
        # Occasional longer break (like human reading)
        if i % 10 == 0:
            time.sleep(random.uniform(20, 40))
        
        # Vary request headers
        headers = {
            'User-Agent': random.choice(USER_AGENTS),
            'Accept-Language': random.choice(['en-US', 'en-GB', 'en']),
        }
        
        response = requests.get(url, headers=headers)
```

## Advanced Techniques

### 1. Scraping Single Page Applications (SPAs)

**Problem**: React/Vue/Angular apps with client-side routing

**Solution**: Intercept API calls

```python
from playwright.sync_api import sync_playwright
import json

def scrape_spa_api_data(url):
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        
        api_responses = []
        
        # Intercept API calls
        def handle_response(response):
            if 'api/products' in response.url:
                api_responses.append(response.json())
        
        page.on('response', handle_response)
        
        page.goto(url)
        page.wait_for_load_state('networkidle')
        
        browser.close()
        
        return api_responses
```

### 2. Handling Infinite Scroll

**Selenium approach**:
```python
from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.get(url)

SCROLL_PAUSE_TIME = 2
last_height = driver.execute_script("return document.body.scrollHeight")

while True:
    # Scroll to bottom
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    
    # Wait for new content
    time.sleep(SCROLL_PAUSE_TIME)
    
    # Calculate new height
    new_height = driver.execute_script("return document.body.scrollHeight")
    
    if new_height == last_height:
        # Reached bottom
        break
    
    last_height = new_height

# Extract all loaded content
products = driver.find_elements(By.CSS_SELECTOR, 'div.product')
```

**Playwright approach** (more robust):
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto(url)
    
    # Scroll until no new items load
    previous_count = 0
    while True:
        # Count items
        current_count = page.locator('div.product').count()
        
        if current_count == previous_count:
            break
        
        previous_count = current_count
        
        # Scroll to last item
        page.locator('div.product').last.scroll_into_view_if_needed()
        page.wait_for_timeout(1000)
    
    # Extract data
    products = page.locator('div.product').all()
```

### 3. Dealing with Shadow DOM

**Problem**: Content inside Shadow DOM not accessible

**Solution**: Use JavaScript to access Shadow DOM

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get(url)

# Access shadow root
shadow_host = driver.find_element(By.CSS_SELECTOR, 'my-custom-element')
shadow_root = driver.execute_script('return arguments[0].shadowRoot', shadow_host)

# Find element inside shadow DOM
element = shadow_root.find_element(By.CSS_SELECTOR, 'div.content')
text = element.text
```

### 4. Handling Dynamic URLs (AJAX Pagination)

**Problem**: URLs don't change but content does

**Solution**: Monitor network requests

```python
from playwright.sync_api import sync_playwright
import json

def scrape_ajax_pagination(base_url):
    all_data = []
    
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        
        # Capture XHR responses
        def handle_response(response):
            if 'api/products' in response.url and response.status == 200:
                try:
                    data = response.json()
                    all_data.append(data)
                except:
                    pass
        
        page.on('response', handle_response)
        
        page.goto(base_url)
        
        # Click through pagination
        for i in range(10):  # 10 pages
            page.click('button.next-page')
            page.wait_for_timeout(1000)
        
        browser.close()
    
    return all_data
```

### 5. Scraping Behind Login

**Selenium with persistent session**:
```python
from selenium import webdriver
import pickle
import os

def login_and_save_cookies(username, password):
    driver = webdriver.Chrome()
    driver.get('https://example.com/login')
    
    # Login
    driver.find_element(By.NAME, 'username').send_keys(username)
    driver.find_element(By.NAME, 'password').send_keys(password)
    driver.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()
    
    # Wait for login
    time.sleep(3)
    
    # Save cookies
    pickle.dump(driver.get_cookies(), open('cookies.pkl', 'wb'))
    driver.quit()

def scrape_with_saved_cookies(url):
    driver = webdriver.Chrome()
    driver.get('https://example.com')
    
    # Load cookies
    if os.path.exists('cookies.pkl'):
        cookies = pickle.load(open('cookies.pkl', 'rb'))
        for cookie in cookies:
            driver.add_cookie(cookie)
    
    # Now navigate to protected page
    driver.get(url)
    
    # Scrape data
    data = driver.find_element(By.CSS_SELECTOR, 'div.protected-content').text
    
    driver.quit()
    return data
```

### 6. Handling WebSockets

**Problem**: Real-time data via WebSockets

**Solution**: Use Playwright to intercept WebSocket messages

```python
from playwright.sync_api import sync_playwright
import json

def scrape_websocket_data(url):
    messages = []
    
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        
        # Intercept WebSocket messages
        def handle_websocket(ws):
            ws.on('framereceived', lambda payload: messages.append(payload))
        
        page.on('websocket', handle_websocket)
        
        page.goto(url)
        page.wait_for_timeout(10000)  # Wait 10 seconds
        
        browser.close()
    
    return messages
```

## Debugging Techniques

### 1. Save Failed Responses

```python
import requests
from pathlib import Path

def save_failed_response(response, identifier):
    """Save HTML for debugging"""
    if response.status_code != 200:
        Path('failed_responses').mkdir(exist_ok=True)
        filepath = f'failed_responses/{identifier}_{response.status_code}.html'
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(response.text)
        print(f"Saved failed response to {filepath}")
```

### 2. Screenshot on Error

```python
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException

driver = webdriver.Chrome()

try:
    driver.get(url)
    element = driver.find_element(By.CSS_SELECTOR, 'div.not-exist')
except NoSuchElementException:
    driver.save_screenshot('error_screenshot.png')
    with open('error_page_source.html', 'w') as f:
        f.write(driver.page_source)
    raise
finally:
    driver.quit()
```

### 3. Log All Requests

```python
import requests
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

# Log HTTP requests
import http.client as http_client
http_client.HTTPConnection.debuglevel = 1

# Now all requests are logged
response = requests.get(url)
```

## Performance Optimization

### 1. Concurrent Scraping

**Using asyncio with aiohttp**:
```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def scrape_multiple(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        pages = await asyncio.gather(*tasks)
        
        results = []
        for html in pages:
            soup = BeautifulSoup(html, 'lxml')
            # Extract data
            results.append(soup.select('h1')[0].text)
        
        return results

# Run
urls = ['https://example.com/page1', 'https://example.com/page2']
results = asyncio.run(scrape_multiple(urls))
```

**Using ThreadPoolExecutor**:
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

def scrape_url(url):
    response = requests.get(url)
    # Process response
    return response.status_code

urls = ['https://example.com/page' + str(i) for i in range(100)]

with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(scrape_url, url): url for url in urls}
    
    for future in as_completed(futures):
        url = futures[future]
        try:
            status = future.result()
            print(f"{url}: {status}")
        except Exception as e:
            print(f"{url} generated an exception: {e}")
```

### 2. Request Caching

```python
import requests_cache

# Install cache
requests_cache.install_cache(
    'scraping_cache',
    backend='sqlite',
    expire_after=3600  # 1 hour
)

# Now requests are cached
response = requests.get(url)  # Makes request
response = requests.get(url)  # Uses cache
```

### 3. Selective Content Download

```python
# Don't download images, CSS, JS
from selenium import webdriver

chrome_options = webdriver.ChromeOptions()
prefs = {
    'profile.managed_default_content_settings.images': 2,  # Disable images
    'profile.managed_default_content_settings.stylesheets': 2,  # Disable CSS
}
chrome_options.add_experimental_option('prefs', prefs)

driver = webdriver.Chrome(options=chrome_options)
```

## Legal and Ethical Considerations

### Best Practices Checklist

- [ ] Check and respect robots.txt
- [ ] Implement rate limiting (1-2 seconds minimum)
- [ ] Use descriptive User-Agent with contact info
- [ ] Don't scrape personal/private data
- [ ] Read and comply with website Terms of Service
- [ ] Cache responses to minimize requests
- [ ] Handle errors gracefully without retry storms
- [ ] Consider using official API if available
- [ ] Don't bypass authentication or paywalls
- [ ] Respect copyright and intellectual property

### robots.txt Checker

```python
import requests
from urllib.parse import urlparse, urljoin
from urllib.robotparser import RobotFileParser

def can_fetch(url, user_agent='*'):
    """Check if URL can be scraped according to robots.txt"""
    parsed = urlparse(url)
    robots_url = f"{parsed.scheme}://{parsed.netloc}/robots.txt"
    
    rp = RobotFileParser()
    rp.set_url(robots_url)
    rp.read()
    
    return rp.can_fetch(user_agent, url)

# Usage
if can_fetch('https://example.com/products'):
    # Safe to scrape
    scrape_page('https://example.com/products')
else:
    print("robots.txt disallows scraping this page")
```
