---
name: web-scraping
description: Provides guidance for extracting data from websites using BeautifulSoup, Scrapy, and Selenium, including HTML parsing, dynamic content handling, pagination, rate limiting, and ethical scraping practices for data collection workflows
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Acquisition, Web Scraping, BeautifulSoup, Scrapy, Selenium, Python, HTML Parsing]
dependencies: [beautifulsoup4>=4.12.0, requests>=2.31.0, lxml>=4.9.0, scrapy>=2.11.0, selenium>=4.15.0]
---

# Web Scraping

This skill provides expert guidance for extracting data from websites using Python's most powerful scraping tools: BeautifulSoup for simple parsing, Scrapy for large-scale scraping projects, and Selenium for JavaScript-heavy sites. Learn ethical scraping practices, anti-bot countermeasures, and data extraction patterns.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Installation & Setup](#installation--setup)
- [Basic Workflows](#basic-workflows)
- [Advanced Patterns](#advanced-patterns)
- [When to Use vs Alternatives](#when-to-use-vs-alternatives)
- [Common Issues & Solutions](#common-issues--solutions)
- [Ethical Scraping Guidelines](#ethical-scraping-guidelines)

## Core Concepts

### Web Scraping Tools Comparison

| Tool | Use Case | Learning Curve | Speed | JavaScript Support |
|------|----------|----------------|-------|-------------------|
| BeautifulSoup | Simple parsing, one-off tasks | Easy ⭐⭐⭐⭐⭐ | Moderate | ❌ |
| Scrapy | Large-scale projects, crawling | Medium ⭐⭐⭐ | Fast ⭐⭐⭐⭐⭐ | Via middleware |
| Selenium | Dynamic sites, automation | Medium ⭐⭐⭐ | Slow ⭐⭐ | ✅ Full |
| Requests-HTML | Simple dynamic content | Easy ⭐⭐⭐⭐ | Moderate | ✅ Basic |

### Key Components

1. **HTTP Requests**: Fetching web pages
2. **HTML Parsing**: Extracting data from markup
3. **Selectors**: CSS/XPath for targeting elements
4. **Rate Limiting**: Respecting server resources
5. **Error Handling**: Dealing with failures gracefully

### robots.txt and Legal Considerations

Always check `robots.txt` before scraping:
```python
import requests

robots = requests.get('https://example.com/robots.txt')
print(robots.text)
```

**Ethical Guidelines**:
- ✅ Respect robots.txt directives
- ✅ Implement rate limiting (1-2 requests/second max)
- ✅ Use descriptive User-Agent
- ✅ Cache responses when possible
- ❌ Don't scrape personal data without consent
- ❌ Don't overwhelm servers

## Installation & Setup

### Basic Installation

```bash
# BeautifulSoup with requests
pip install beautifulsoup4 requests lxml html5lib

# Scrapy
pip install scrapy

# Selenium with browser drivers
pip install selenium

# Install browser drivers
# Chrome
pip install webdriver-manager

# Alternative: requests-html (simple JavaScript rendering)
pip install requests-html
```

### Browser Driver Setup for Selenium

```bash
# Option 1: Use webdriver-manager (automatic)
pip install webdriver-manager

# Option 2: Manual download
# Chrome: https://chromedriver.chromium.org/
# Firefox: https://github.com/mozilla/geckodriver/releases
```

## Basic Workflows

### Workflow 1: Simple Scraping with BeautifulSoup

**Use Case**: Extract data from static HTML pages

**Steps**:

1. **Fetch webpage**:
```python
import requests
from bs4 import BeautifulSoup

# Make request with proper headers
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
}

url = 'https://example.com/products'
response = requests.get(url, headers=headers)

# Check response
if response.status_code == 200:
    print(f"Successfully fetched {url}")
else:
    print(f"Failed: {response.status_code}")
```

2. **Parse HTML**:
```python
# Create BeautifulSoup object
soup = BeautifulSoup(response.content, 'lxml')

# Alternative parsers:
# soup = BeautifulSoup(response.content, 'html.parser')  # Built-in
# soup = BeautifulSoup(response.content, 'html5lib')     # More lenient
```

3. **Extract data using CSS selectors**:
```python
# Find all product cards
products = soup.select('div.product-card')

# Extract data from each product
data = []
for product in products:
    # Find elements within each product
    name = product.select_one('h2.product-name')
    price = product.select_one('span.price')
    rating = product.select_one('div.rating')
    
    data.append({
        'name': name.text.strip() if name else None,
        'price': price.text.strip() if price else None,
        'rating': rating.text.strip() if rating else None
    })

print(f"Extracted {len(data)} products")
```

4. **Convert to DataFrame**:
```python
import pandas as pd

df = pd.DataFrame(data)
print(df.head())

# Clean data
df['price'] = df['price'].str.replace('$', '').str.replace(',', '').astype(float)
df['rating'] = df['rating'].str.extract(r'([\d.]+)').astype(float)
```

5. **Save results**:
```python
# Save to CSV
df.to_csv('products.csv', index=False)

# Or JSON
df.to_json('products.json', orient='records', indent=2)
```

**Checklist**:
- [ ] Check robots.txt for permission
- [ ] Use appropriate User-Agent header
- [ ] Verify response status code
- [ ] Handle missing elements gracefully
- [ ] Clean and validate extracted data
- [ ] Implement rate limiting for multiple pages

### Workflow 2: Large-Scale Scraping with Scrapy

**Use Case**: Crawl multiple pages or entire websites efficiently

**Steps**:

1. **Create Scrapy project**:
```bash
scrapy startproject myproject
cd myproject
scrapy genspider products example.com
```

2. **Define items** (myproject/items.py):
```python
import scrapy

class ProductItem(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    rating = scrapy.Field()
    url = scrapy.Field()
    scraped_at = scrapy.Field()
```

3. **Write spider** (myproject/spiders/products.py):
```python
import scrapy
from myproject.items import ProductItem
from datetime import datetime

class ProductsSpider(scrapy.Spider):
    name = 'products'
    allowed_domains = ['example.com']
    start_urls = ['https://example.com/products']
    
    # Custom settings
    custom_settings = {
        'DOWNLOAD_DELAY': 1,  # 1 second between requests
        'CONCURRENT_REQUESTS': 4,
        'ROBOTSTXT_OBEY': True
    }
    
    def parse(self, response):
        # Extract products from current page
        for product in response.css('div.product-card'):
            item = ProductItem()
            item['name'] = product.css('h2.product-name::text').get()
            item['price'] = product.css('span.price::text').get()
            item['rating'] = product.css('div.rating::text').get()
            item['url'] = response.urljoin(product.css('a::attr(href)').get())
            item['scraped_at'] = datetime.now().isoformat()
            
            yield item
        
        # Follow pagination
        next_page = response.css('a.next-page::attr(href)').get()
        if next_page:
            yield response.follow(next_page, callback=self.parse)
```

4. **Configure settings** (myproject/settings.py):
```python
# Obey robots.txt
ROBOTSTXT_OBEY = True

# User agent
USER_AGENT = 'MyBot/1.0 (contact@example.com)'

# Rate limiting
DOWNLOAD_DELAY = 1
CONCURRENT_REQUESTS_PER_DOMAIN = 4

# AutoThrottle (adaptive rate limiting)
AUTOTHROTTLE_ENABLED = True
AUTOTHROTTLE_START_DELAY = 1
AUTOTHROTTLE_MAX_DELAY = 10
AUTOTHROTTLE_TARGET_CONCURRENCY = 2.0

# Retry on failure
RETRY_TIMES = 3
RETRY_HTTP_CODES = [500, 502, 503, 504, 408, 429]
```

5. **Run spider**:
```bash
# Output to JSON
scrapy crawl products -o products.json

# Output to CSV
scrapy crawl products -o products.csv

# Output to JSON Lines (better for large datasets)
scrapy crawl products -o products.jsonl
```

**Checklist**:
- [ ] Define clear Item schema
- [ ] Set appropriate download delays
- [ ] Enable ROBOTSTXT_OBEY
- [ ] Implement pagination handling
- [ ] Add error handling in parse methods
- [ ] Test on small subset first

### Workflow 3: Scraping Dynamic Sites with Selenium

**Use Case**: Extract data from JavaScript-rendered pages

**Steps**:

1. **Setup Selenium with browser**:
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options

# Configure Chrome options
chrome_options = Options()
chrome_options.add_argument('--headless')  # Run without GUI
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64)')

# Initialize driver
service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=chrome_options)
```

2. **Navigate and wait for content**:
```python
url = 'https://example.com/dynamic-content'
driver.get(url)

# Wait for specific element to load
wait = WebDriverWait(driver, 10)
wait.until(
    EC.presence_of_element_located((By.CLASS_NAME, 'product-card'))
)

# Alternative: wait for JavaScript to complete
driver.execute_script("return document.readyState") == "complete"
```

3. **Handle infinite scroll**:
```python
import time

# Scroll to bottom multiple times
SCROLL_PAUSE_TIME = 2
last_height = driver.execute_script("return document.body.scrollHeight")

while True:
    # Scroll down
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    
    # Wait for new content to load
    time.sleep(SCROLL_PAUSE_TIME)
    
    # Check if reached bottom
    new_height = driver.execute_script("return document.body.scrollHeight")
    if new_height == last_height:
        break
    last_height = new_height
```

4. **Extract data**:
```python
# Find elements
products = driver.find_elements(By.CSS_SELECTOR, 'div.product-card')

data = []
for product in products:
    # Extract text from elements
    name = product.find_element(By.CSS_SELECTOR, 'h2.product-name').text
    price = product.find_element(By.CSS_SELECTOR, 'span.price').text
    
    # Extract attributes
    link = product.find_element(By.CSS_SELECTOR, 'a').get_attribute('href')
    image = product.find_element(By.CSS_SELECTOR, 'img').get_attribute('src')
    
    data.append({
        'name': name,
        'price': price,
        'link': link,
        'image': image
    })

print(f"Extracted {len(data)} products")
```

5. **Convert to DataFrame and cleanup**:
```python
import pandas as pd

df = pd.DataFrame(data)

# Clean up
driver.quit()

# Save data
df.to_csv('dynamic_products.csv', index=False)
```

**Checklist**:
- [ ] Use headless mode for production
- [ ] Implement explicit waits (not time.sleep)
- [ ] Handle StaleElementReferenceException
- [ ] Close driver after use
- [ ] Consider using Selenium Grid for scale
- [ ] Use browser profiles to persist cookies/sessions

## Advanced Patterns

### Handling Pagination

**Pattern 1: Next button**:
```python
from bs4 import BeautifulSoup
import requests

def scrape_all_pages(base_url):
    all_data = []
    current_url = base_url
    
    while current_url:
        response = requests.get(current_url)
        soup = BeautifulSoup(response.content, 'lxml')
        
        # Extract data from current page
        data = extract_page_data(soup)
        all_data.extend(data)
        
        # Find next page link
        next_link = soup.select_one('a.next-page')
        current_url = next_link['href'] if next_link else None
        
        # Rate limiting
        time.sleep(1)
    
    return all_data
```

**Pattern 2: Page numbers**:
```python
def scrape_paginated(base_url, max_pages=10):
    all_data = []
    
    for page in range(1, max_pages + 1):
        url = f"{base_url}?page={page}"
        response = requests.get(url)
        
        # Stop if page doesn't exist
        if response.status_code != 200:
            break
        
        soup = BeautifulSoup(response.content, 'lxml')
        data = extract_page_data(soup)
        
        # Stop if no data found
        if not data:
            break
        
        all_data.extend(data)
        time.sleep(1)
    
    return all_data
```

### Session Management and Cookies

```python
import requests

# Create session to persist cookies
session = requests.Session()

# Set headers
session.headers.update({
    'User-Agent': 'Mozilla/5.0',
    'Accept-Language': 'en-US,en;q=0.9'
})

# Login example
login_data = {'username': 'user', 'password': 'pass'}
session.post('https://example.com/login', data=login_data)

# Now session has cookies, can access protected pages
response = session.get('https://example.com/protected')

# Save cookies for later
import pickle
with open('cookies.pkl', 'wb') as f:
    pickle.dump(session.cookies, f)

# Load cookies
with open('cookies.pkl', 'rb') as f:
    session.cookies.update(pickle.load(f))
```

### Rate Limiting and Respectful Scraping

```python
import time
from functools import wraps

def rate_limit(delay=1):
    """Decorator to add delay between function calls"""
    def decorator(func):
        last_called = [0.0]
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < delay:
                time.sleep(delay - elapsed)
            
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        
        return wrapper
    return decorator

# Usage
@rate_limit(delay=2)  # 2 seconds between calls
def fetch_url(url):
    return requests.get(url)

# Alternative: Using RateLimiter library
from ratelimit import limits, sleep_and_retry

@sleep_and_retry
@limits(calls=10, period=60)  # 10 calls per minute
def fetch_url_limited(url):
    return requests.get(url)
```

### Error Handling and Retries

```python
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

def create_session_with_retries():
    """Create session with automatic retries"""
    session = requests.Session()
    
    # Configure retries
    retry_strategy = Retry(
        total=3,                    # Total retries
        backoff_factor=1,           # Wait 1, 2, 4 seconds
        status_forcelist=[429, 500, 502, 503, 504],
        allowed_methods=["GET", "POST"]
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    
    return session

# Usage
session = create_session_with_retries()
try:
    response = session.get('https://example.com')
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    print(f"Failed after retries: {e}")
```

### Extracting Data from Tables

```python
from bs4 import BeautifulSoup
import pandas as pd

def scrape_table(url, table_selector='table'):
    """Scrape HTML table to DataFrame"""
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'lxml')
    
    # Find table
    table = soup.select_one(table_selector)
    
    # Extract headers
    headers = [th.text.strip() for th in table.select('thead th')]
    
    # Extract rows
    rows = []
    for tr in table.select('tbody tr'):
        row = [td.text.strip() for td in tr.select('td')]
        rows.append(row)
    
    # Create DataFrame
    df = pd.DataFrame(rows, columns=headers)
    return df

# Or use pandas directly
df = pd.read_html('https://example.com/table.html')[0]  # First table
```

## When to Use vs Alternatives

### Use Web Scraping When:
- ✅ No official API available
- ✅ API has severe rate limits
- ✅ Need historical data not in API
- ✅ Data only available on web interface
- ✅ Public data aggregation for research

### Consider Alternatives When:
- ❌ **Official API exists**: Always prefer official APIs
- ❌ **Data available in downloads**: Check for CSV/JSON exports
- ❌ **RSS/Atom feeds**: Use feed readers instead
- ❌ **Paid data services**: Consider cost vs effort
- ❌ **Legal restrictions**: Respect ToS and copyright

### Tool Selection Guide:

**Use BeautifulSoup when**:
- Quick one-off scraping tasks
- Static HTML pages
- Simple data extraction
- Learning web scraping

**Use Scrapy when**:
- Large-scale crawling (1000+ pages)
- Need structured pipeline
- Multiple concurrent spiders
- Long-running scraping projects

**Use Selenium when**:
- JavaScript-rendered content
- Need to interact with page (clicks, forms)
- AJAX-heavy sites
- Login/authentication required

## Common Issues & Solutions

### Issue 1: 403 Forbidden or 429 Too Many Requests

**Problem**: Server blocking requests

```python
# ❌ Gets blocked
response = requests.get(url)  # Default user-agent
```

**Solution**: Use proper headers and rate limiting

```python
# ✅ Better approach
import time
import random

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Connection': 'keep-alive',
}

session = requests.Session()
session.headers.update(headers)

# Add random delay
time.sleep(random.uniform(1, 3))

response = session.get(url)
```

### Issue 2: JavaScript Content Not Loading

**Problem**: Page loads but data is empty

**Solution**: Use Selenium or requests-html

```python
# Method 1: Selenium
from selenium import webdriver

driver = webdriver.Chrome()
driver.get(url)
time.sleep(2)  # Wait for JavaScript
html = driver.page_source
driver.quit()

# Parse with BeautifulSoup
soup = BeautifulSoup(html, 'lxml')

# Method 2: requests-html (simpler)
from requests_html import HTMLSession

session = HTMLSession()
response = session.get(url)
response.html.render()  # Executes JavaScript
data = response.html.find('div.product-card')
```

### Issue 3: Element Not Found

**Problem**: CSS selector returns None

```python
# ❌ Crashes if element missing
name = product.select_one('h2.name').text
```

**Solution**: Handle missing elements gracefully

```python
# ✅ Safe extraction
name_elem = product.select_one('h2.name')
name = name_elem.text.strip() if name_elem else 'N/A'

# Or use get() method (returns None if not found)
name = product.select_one('h2.name')
if name:
    name = name.text.strip()
```

### Issue 4: Encoding Issues

**Problem**: Special characters appear as gibberish

**Solution**: Specify encoding

```python
import requests

response = requests.get(url)

# Detect encoding
print(f"Detected encoding: {response.encoding}")

# Force UTF-8 if needed
response.encoding = 'utf-8'

# Then parse
soup = BeautifulSoup(response.text, 'lxml')
```

### Issue 5: Selenium WebDriver Crashes

**Problem**: Browser crashes or hangs

**Solution**: Use robust configuration

```python
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
options.add_argument('--disable-gpu')
options.add_argument('--window-size=1920,1080')

# Set page load timeout
driver = webdriver.Chrome(options=options)
driver.set_page_load_timeout(30)

try:
    driver.get(url)
except TimeoutException:
    print("Page load timed out")
finally:
    driver.quit()
```

## Ethical Scraping Guidelines

### Best Practices

1. **Check robots.txt**:
```python
import requests

robots_url = 'https://example.com/robots.txt'
robots = requests.get(robots_url)
print(robots.text)
```

2. **Respect rate limits**:
```python
# 1-2 seconds between requests minimum
time.sleep(1)

# Better: use exponential backoff
import random
time.sleep(random.uniform(1, 3))
```

3. **Use descriptive User-Agent**:
```python
headers = {
    'User-Agent': 'MyResearchBot/1.0 (contact@example.com; Research purpose)'
}
```

4. **Handle errors gracefully**:
```python
try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
    # Don't retry immediately
    time.sleep(60)
```

5. **Cache responses**:
```python
import requests_cache

# Enable caching
requests_cache.install_cache('scraping_cache', expire_after=3600)

# Now requests are cached
response = requests.get(url)
```

### Legal Considerations

- ✅ **Public data**: Generally OK to scrape public information
- ⚠️ **Terms of Service**: Check website ToS
- ⚠️ **Copyright**: Respect intellectual property
- ❌ **Personal data**: GDPR/privacy laws apply
- ❌ **Bypass paywalls**: Usually illegal
- ❌ **CAPTCHA circumvention**: Violates ToS

## Additional Resources

- **Selector Guides**: [references/selector-guide.md](references/selector-guide.md)
- **Advanced Scrapy**: [references/scrapy-advanced.md](references/scrapy-advanced.md)
- **Anti-Bot Techniques**: [references/anti-bot-strategies.md](references/anti-bot-strategies.md)

## Summary

This skill covered:
- ✅ BeautifulSoup for simple HTML parsing
- ✅ Scrapy for large-scale crawling
- ✅ Selenium for dynamic content
- ✅ Rate limiting and respectful scraping
- ✅ Error handling and retries
- ✅ Ethical guidelines and legal considerations

**Key Takeaways**:
1. Always check robots.txt and respect it
2. Implement rate limiting (1-2 seconds minimum)
3. Use proper User-Agent headers
4. Handle errors and missing data gracefully
5. Choose the right tool for the job (BeautifulSoup/Scrapy/Selenium)
6. Cache responses to avoid redundant requests
