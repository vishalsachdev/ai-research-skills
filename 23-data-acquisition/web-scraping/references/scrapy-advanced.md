# Advanced Scrapy Guide

Advanced patterns, pipelines, and production configurations for Scrapy web scraping framework.

## Scrapy Architecture

### Components

1. **Spider**: Defines how to scrape a site
2. **Item**: Data structure for scraped data
3. **Pipeline**: Process and store items
4. **Middleware**: Customize request/response handling
5. **Settings**: Configuration

### Request Flow

```
Spider → Request → Downloader Middleware → Downloader → Response
→ Spider Middleware → Spider → Item → Item Pipeline → Storage
```

## Advanced Spider Patterns

### CrawlSpider for Site Crawling

```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from myproject.items import ArticleItem

class NewsSpider(CrawlSpider):
    name = 'news'
    allowed_domains = ['example.com']
    start_urls = ['https://example.com/news']
    
    # Define crawling rules
    rules = (
        # Follow category pages
        Rule(
            LinkExtractor(allow=r'/category/\w+'),
            follow=True
        ),
        # Parse article pages
        Rule(
            LinkExtractor(allow=r'/article/[\w-]+'),
            callback='parse_article',
            follow=False
        ),
        # Follow pagination
        Rule(
            LinkExtractor(restrict_css='a.next-page'),
            follow=True
        ),
    )
    
    def parse_article(self, response):
        item = ArticleItem()
        item['title'] = response.css('h1.article-title::text').get()
        item['content'] = response.css('div.article-body::text').getall()
        item['author'] = response.css('span.author::text').get()
        item['published'] = response.css('time::attr(datetime)').get()
        item['url'] = response.url
        
        yield item
```

### Spider with Login

```python
import scrapy

class LoginSpider(scrapy.Spider):
    name = 'login_spider'
    start_urls = ['https://example.com/login']
    
    def parse(self, response):
        # Extract CSRF token if present
        csrf_token = response.css('input[name="csrf_token"]::attr(value)').get()
        
        # Submit login form
        return scrapy.FormRequest.from_response(
            response,
            formdata={
                'username': 'myuser',
                'password': 'mypass',
                'csrf_token': csrf_token
            },
            callback=self.after_login
        )
    
    def after_login(self, response):
        # Check if login succeeded
        if b'Logout' in response.body:
            self.logger.info('Login successful')
            
            # Start scraping protected pages
            yield scrapy.Request(
                'https://example.com/protected',
                callback=self.parse_protected
            )
        else:
            self.logger.error('Login failed')
    
    def parse_protected(self, response):
        # Scrape data from protected pages
        pass
```

### Handling Pagination

**Method 1: Follow next links**:
```python
def parse(self, response):
    # Extract items from current page
    for item in response.css('div.item'):
        yield {
            'title': item.css('h2::text').get(),
            'price': item.css('span.price::text').get()
        }
    
    # Follow next page link
    next_page = response.css('a.next::attr(href)').get()
    if next_page:
        yield response.follow(next_page, callback=self.parse)
```

**Method 2: Iterate page numbers**:
```python
def start_requests(self):
    base_url = 'https://example.com/products?page={}'
    for page in range(1, 101):  # 100 pages
        yield scrapy.Request(
            base_url.format(page),
            callback=self.parse,
            meta={'page': page}
        )

def parse(self, response):
    page = response.meta['page']
    self.logger.info(f'Scraping page {page}')
    
    # Extract items
    items = response.css('div.item')
    
    # Stop if no items (reached end)
    if not items:
        self.logger.info(f'No items on page {page}, stopping')
        return
    
    for item in items:
        yield {'title': item.css('h2::text').get()}
```

### Passing Data Between Callbacks

```python
def parse(self, response):
    # Parse list page
    for product in response.css('div.product'):
        product_url = product.css('a::attr(href)').get()
        
        # Pass data to next callback via meta
        yield response.follow(
            product_url,
            callback=self.parse_product,
            meta={
                'category': response.meta.get('category'),
                'list_price': product.css('span.price::text').get()
            }
        )

def parse_product(self, response):
    # Access data from previous callback
    category = response.meta['category']
    list_price = response.meta['list_price']
    
    yield {
        'title': response.css('h1::text').get(),
        'category': category,
        'list_price': list_price,
        'detail_price': response.css('span.price::text').get()
    }
```

## Items and Item Loaders

### Defining Items

```python
import scrapy
from scrapy.loader import ItemLoader
from itemloaders.processors import TakeFirst, MapCompose, Join
from w3lib.html import remove_tags

class ProductItem(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    description = scrapy.Field()
    images = scrapy.Field()
    availability = scrapy.Field()
    url = scrapy.Field()
```

### Using Item Loaders

```python
from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, MapCompose, Join
import re

class ProductLoader(ItemLoader):
    default_output_processor = TakeFirst()
    
    # Price: remove $, commas, convert to float
    price_in = MapCompose(
        lambda x: x.strip(),
        lambda x: re.sub(r'[,$]', '', x)
    )
    price_out = TakeFirst()
    
    # Description: remove HTML, join paragraphs
    description_in = MapCompose(remove_tags, lambda x: x.strip())
    description_out = Join('\n')
    
    # Images: collect all
    images_out = lambda x: x  # Don't take first, keep all

# Usage in spider
def parse_product(self, response):
    loader = ProductLoader(item=ProductItem(), response=response)
    
    loader.add_css('name', 'h1.product-name::text')
    loader.add_css('price', 'span.price::text')
    loader.add_css('description', 'div.description p::text')
    loader.add_css('images', 'img.product-image::attr(src)')
    loader.add_value('url', response.url)
    
    return loader.load_item()
```

## Pipelines

### Basic Pipeline

```python
# pipelines.py
import json
from itemadapter import ItemAdapter

class JsonWriterPipeline:
    def open_spider(self, spider):
        self.file = open('items.jsonl', 'w')
    
    def close_spider(self, spider):
        self.file.close()
    
    def process_item(self, item, spider):
        line = json.dumps(ItemAdapter(item).asdict()) + "\n"
        self.file.write(line)
        return item
```

### Data Cleaning Pipeline

```python
import re
from itemadapter import ItemAdapter

class DataCleaningPipeline:
    def process_item(self, item, spider):
        adapter = ItemAdapter(item)
        
        # Clean price
        if adapter.get('price'):
            price = adapter['price']
            price = re.sub(r'[^0-9.]', '', price)
            adapter['price'] = float(price) if price else None
        
        # Normalize text
        if adapter.get('name'):
            adapter['name'] = adapter['name'].strip().title()
        
        # Validate required fields
        if not adapter.get('name'):
            raise DropItem(f"Missing name in {item}")
        
        return item
```

### Database Pipeline

```python
import pymongo
from itemadapter import ItemAdapter

class MongoPipeline:
    collection_name = 'scrapy_items'
    
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )
    
    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]
    
    def close_spider(self, spider):
        self.client.close()
    
    def process_item(self, item, spider):
        self.db[self.collection_name].insert_one(ItemAdapter(item).asdict())
        return item
```

### Duplicate Filter Pipeline

```python
from scrapy.exceptions import DropItem

class DuplicatesPipeline:
    def __init__(self):
        self.ids_seen = set()
    
    def process_item(self, item, spider):
        adapter = ItemAdapter(item)
        
        if adapter['id'] in self.ids_seen:
            raise DropItem(f"Duplicate item found: {item['id']}")
        else:
            self.ids_seen.add(adapter['id'])
            return item
```

### Image Download Pipeline

```python
# Built-in image pipeline
ITEM_PIPELINES = {
    'scrapy.pipelines.images.ImagesPipeline': 1,
}

IMAGES_STORE = '/path/to/images'
IMAGES_URLS_FIELD = 'image_urls'
IMAGES_RESULT_FIELD = 'images'
IMAGES_THUMBS = {
    'small': (50, 50),
    'medium': (200, 200),
}

# In spider
def parse_product(self, response):
    yield {
        'name': response.css('h1::text').get(),
        'image_urls': response.css('img::attr(src)').getall()
    }
```

## Middleware

### Custom Downloader Middleware

```python
# middlewares.py
import random
from scrapy import signals

class RandomUserAgentMiddleware:
    def __init__(self, user_agents):
        self.user_agents = user_agents
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            user_agents=crawler.settings.getlist('USER_AGENTS')
        )
    
    def process_request(self, request, spider):
        request.headers['User-Agent'] = random.choice(self.user_agents)

class ProxyMiddleware:
    def __init__(self, proxies):
        self.proxies = proxies
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(proxies=crawler.settings.getlist('PROXIES'))
    
    def process_request(self, request, spider):
        request.meta['proxy'] = random.choice(self.proxies)
```

### Retry Middleware

```python
from scrapy.downloadermiddlewares.retry import RetryMiddleware
from scrapy.utils.response import response_status_message

class CustomRetryMiddleware(RetryMiddleware):
    def process_response(self, request, response, spider):
        # Retry on specific status codes
        if response.status in [500, 502, 503, 504, 408, 429]:
            reason = response_status_message(response.status)
            return self._retry(request, reason, spider) or response
        
        # Retry if specific text in response
        if b'temporarily unavailable' in response.body:
            return self._retry(request, 'temp_unavailable', spider) or response
        
        return response
```

## Advanced Settings

### Production Settings

```python
# settings.py

# Obey robots.txt
ROBOTSTXT_OBEY = True

# Configure maximum concurrent requests
CONCURRENT_REQUESTS = 16
CONCURRENT_REQUESTS_PER_DOMAIN = 4
CONCURRENT_REQUESTS_PER_IP = 4

# Download delay (seconds)
DOWNLOAD_DELAY = 1
DOWNLOAD_TIMEOUT = 30

# AutoThrottle
AUTOTHROTTLE_ENABLED = True
AUTOTHROTTLE_START_DELAY = 1
AUTOTHROTTLE_MAX_DELAY = 10
AUTOTHROTTLE_TARGET_CONCURRENCY = 2.0
AUTOTHROTTLE_DEBUG = False

# HTTP caching
HTTPCACHE_ENABLED = True
HTTPCACHE_EXPIRATION_SECS = 3600
HTTPCACHE_DIR = 'httpcache'
HTTPCACHE_IGNORE_HTTP_CODES = [500, 502, 503, 504, 408]

# User agent
USER_AGENT = 'MyBot/1.0 (+http://example.com/bot)'

# Retry configuration
RETRY_ENABLED = True
RETRY_TIMES = 3
RETRY_HTTP_CODES = [500, 502, 503, 504, 408, 429]

# Enable pipelines
ITEM_PIPELINES = {
    'myproject.pipelines.DataCleaningPipeline': 100,
    'myproject.pipelines.DuplicatesPipeline': 200,
    'myproject.pipelines.MongoPipeline': 300,
}

# Enable middlewares
DOWNLOADER_MIDDLEWARES = {
    'myproject.middlewares.RandomUserAgentMiddleware': 400,
    'myproject.middlewares.ProxyMiddleware': 410,
}
```

### Environment-Specific Settings

```python
# settings_dev.py
from .settings import *

DOWNLOAD_DELAY = 0.5
CONCURRENT_REQUESTS = 8
HTTPCACHE_ENABLED = True
LOG_LEVEL = 'DEBUG'

# settings_prod.py
from .settings import *

DOWNLOAD_DELAY = 2
CONCURRENT_REQUESTS = 32
HTTPCACHE_ENABLED = False
LOG_LEVEL = 'INFO'

# Use with:
# scrapy crawl myspider --set=SETTINGS_MODULE=myproject.settings_prod
```

## Scrapy Shell

### Interactive Debugging

```bash
# Start shell with URL
scrapy shell 'https://example.com/products'

# In shell
>>> response.css('h1::text').get()
'Products Page'

>>> response.xpath('//div[@class="product"]').getall()

>>> fetch('https://example.com/other-page')  # Load different URL

>>> view(response)  # Open in browser
```

### Shell Commands

```python
# In scrapy shell

# Test selectors
response.css('div.product')
response.xpath('//div[@class="product"]')

# Extract text
response.css('h1::text').get()
response.css('h1::text').getall()

# Extract attributes
response.css('a::attr(href)').get()

# Follow links
fetch(response.urljoin('/products/page2'))

# Test Item Loaders
from myproject.items import ProductItem
from myproject.loaders import ProductLoader

loader = ProductLoader(item=ProductItem(), response=response)
loader.add_css('name', 'h1::text')
loader.load_item()
```

## Deployment

### Scrapy Cloud (ScrapingHub)

```bash
# Install shub
pip install shub

# Login
shub login

# Deploy
shub deploy
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Run spider
CMD ["scrapy", "crawl", "myspider", "-o", "output.json"]
```

```bash
# Build and run
docker build -t myscrapyproject .
docker run -v $(pwd)/output:/app/output myscrapyproject
```

### Scheduled Scraping with Cron

```bash
# crontab -e

# Run every day at 2am
0 2 * * * cd /path/to/project && scrapy crawl myspider -o output_$(date +\%Y\%m\%d).json

# Run every hour
0 * * * * cd /path/to/project && scrapy crawl myspider
```

## Performance Optimization

### 1. Use Proper Selectors

```python
# Slow: XPath is slower than CSS
response.xpath('//div[@class="product"]//h2/text()')

# Fast: CSS selectors
response.css('div.product h2::text')
```

### 2. Limit Response Size

```python
# settings.py
DOWNLOAD_MAXSIZE = 1048576  # 1MB max
DOWNLOAD_WARNSIZE = 524288  # Warn at 512KB
```

### 3. Disable Unnecessary Features

```python
# Disable cookies if not needed
COOKIES_ENABLED = False

# Disable redirects if not needed
REDIRECT_ENABLED = False

# Disable retry if not needed
RETRY_ENABLED = False
```

### 4. Use HTTP Cache

```python
HTTPCACHE_ENABLED = True
HTTPCACHE_EXPIRATION_SECS = 86400  # 1 day
HTTPCACHE_DIR = 'httpcache'
```

### 5. Optimize Database Writes

```python
# Batch writes instead of one-by-one
class BatchMongoPipeline:
    def __init__(self):
        self.items = []
        self.batch_size = 100
    
    def process_item(self, item, spider):
        self.items.append(ItemAdapter(item).asdict())
        
        if len(self.items) >= self.batch_size:
            self.db[self.collection].insert_many(self.items)
            self.items = []
        
        return item
    
    def close_spider(self, spider):
        if self.items:
            self.db[self.collection].insert_many(self.items)
```

## Monitoring and Logging

### Custom Stats

```python
class MySpider(scrapy.Spider):
    def parse(self, response):
        # Increment custom stat
        self.crawler.stats.inc_value('products_scraped')
        
        # Set custom value
        self.crawler.stats.set_value('last_scraped', datetime.now())
        
        yield {'title': response.css('h1::text').get()}
```

### Email Notifications

```python
# settings.py
MAIL_FROM = 'scraper@example.com'
MAIL_TO = ['admin@example.com']

# Extension for email alerts
EXTENSIONS = {
    'scrapy.extensions.statsmailer.StatsMailer': 500,
}

STATSMAILER_RCPTS = ['admin@example.com']
```

### JSON Logging

```python
import json
import logging

class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'message': record.getMessage(),
            'spider': getattr(record, 'spider', None)
        }
        return json.dumps(log_data)

# In spider
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger = logging.getLogger('myspider')
logger.addHandler(handler)
```
