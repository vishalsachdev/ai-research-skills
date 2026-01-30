# CSS and XPath Selector Guide

Comprehensive guide for targeting HTML elements using CSS selectors and XPath expressions.

## CSS Selectors

### Basic Selectors

**Element selector**:
```python
# Select all <p> tags
soup.select('p')
driver.find_elements(By.CSS_SELECTOR, 'p')
```

**Class selector**:
```python
# Select elements with class="product"
soup.select('.product')
soup.select('div.product')  # div with class="product"

# Multiple classes
soup.select('.product.featured')  # class="product featured"
```

**ID selector**:
```python
# Select element with id="header"
soup.select('#header')
soup.select_one('#header')  # Returns single element
```

**Attribute selectors**:
```python
# Has attribute
soup.select('[href]')  # All elements with href attribute
soup.select('a[href]')  # All <a> tags with href

# Exact match
soup.select('[type="text"]')
soup.select('input[type="text"]')

# Contains
soup.select('[class*="product"]')  # class contains "product"
soup.select('[href*="amazon"]')    # href contains "amazon"

# Starts with
soup.select('[href^="https"]')     # href starts with "https"
soup.select('[class^="btn-"]')     # class starts with "btn-"

# Ends with
soup.select('[href$=".pdf"]')      # href ends with ".pdf"
soup.select('[class$="-large"]')   # class ends with "-large"
```

### Combinators

**Descendant (space)**:
```python
# Any <a> inside div.container
soup.select('div.container a')

# Nested
soup.select('div.product h2.title')
```

**Direct child (>)**:
```python
# <li> that are direct children of <ul>
soup.select('ul > li')

# Not grandchildren
soup.select('div.product > h2')  # Only direct h2 children
```

**Adjacent sibling (+)**:
```python
# <p> immediately after <h2>
soup.select('h2 + p')
```

**General sibling (~)**:
```python
# All <p> after <h2> (same parent)
soup.select('h2 ~ p')
```

### Pseudo-classes

**Positional**:
```python
# First child
soup.select('li:first-child')

# Last child
soup.select('li:last-child')

# Nth child (1-indexed)
soup.select('li:nth-child(2)')    # Second li
soup.select('li:nth-child(odd)')  # Odd-numbered
soup.select('li:nth-child(even)') # Even-numbered
soup.select('li:nth-child(3n)')   # Every 3rd element

# Nth of type
soup.select('p:nth-of-type(2)')   # Second <p> tag
```

**Other pseudo-classes**:
```python
# Not
soup.select('li:not(.active)')    # li without class="active"

# Contains text (BeautifulSoup-specific)
soup.find_all('a', string='Click here')
```

### Advanced Examples

**Complex product card extraction**:
```python
# Complete selector for product data
products = soup.select('div.product-card')

for product in products:
    # Title
    title = product.select_one('h2.product-title a')
    
    # Price (might be in different places)
    price = product.select_one('span.price, div.price-box span')
    
    # Rating
    rating = product.select_one('div.rating span[data-rating]')
    
    # Image
    image = product.select_one('img.product-image')
    
    # Extract data
    data = {
        'title': title.text.strip() if title else None,
        'price': price.text.strip() if price else None,
        'rating': rating['data-rating'] if rating else None,
        'image': image['src'] if image else None
    }
```

**Table scraping**:
```python
# Select all rows in table body
rows = soup.select('table.data-table tbody tr')

for row in rows:
    # Get all cells
    cells = row.select('td')
    
    if len(cells) >= 3:
        data = {
            'col1': cells[0].text.strip(),
            'col2': cells[1].text.strip(),
            'col3': cells[2].text.strip()
        }
```

## XPath Expressions

### Basic XPath

**Element selection**:
```python
# All <p> tags
driver.find_elements(By.XPATH, '//p')

# <p> tags inside <div>
driver.find_elements(By.XPATH, '//div//p')

# Direct children only
driver.find_elements(By.XPATH, '//div/p')
```

**Attribute selection**:
```python
# By class
driver.find_elements(By.XPATH, '//div[@class="product"]')

# By ID
driver.find_element(By.XPATH, '//div[@id="header"]')

# By any attribute
driver.find_elements(By.XPATH, '//a[@href]')
driver.find_elements(By.XPATH, '//input[@type="text"]')
```

**Text content**:
```python
# Contains text
driver.find_elements(By.XPATH, '//a[contains(text(), "Click here")]')

# Exact text match
driver.find_elements(By.XPATH, '//button[text()="Submit"]')

# Starts with
driver.find_elements(By.XPATH, '//div[starts-with(@class, "product-")]')
```

### Advanced XPath

**Multiple conditions (and/or)**:
```python
# AND condition
xpath = '//div[@class="product" and @data-available="true"]'
driver.find_elements(By.XPATH, xpath)

# OR condition
xpath = '//input[@type="text" or @type="email"]'
driver.find_elements(By.XPATH, xpath)
```

**Axes (navigation)**:
```python
# Parent
xpath = '//span[@class="price"]/parent::div'

# Ancestor
xpath = '//span[@class="price"]/ancestor::div[@class="product"]'

# Following sibling
xpath = '//h2[@class="title"]/following-sibling::p'

# Preceding sibling
xpath = '//button/preceding-sibling::input'

# Child
xpath = '//div[@class="container"]/child::p'
```

**Position-based**:
```python
# First element
xpath = '(//div[@class="product"])[1]'

# Last element
xpath = '(//div[@class="product"])[last()]'

# Position greater than
xpath = '//li[position() > 3]'

# Range
xpath = '//li[position() >= 2 and position() <= 5]'
```

**Contains and string functions**:
```python
# Class contains (for multiple classes)
xpath = '//div[contains(@class, "product")]'

# Multiple class check
xpath = '//div[contains(@class, "product") and contains(@class, "featured")]'

# Normalize space (remove extra whitespace)
xpath = '//p[normalize-space(text())="Hello World"]'

# String length
xpath = '//input[string-length(@value) > 10]'
```

### XPath vs CSS Selector Comparison

| Task | CSS Selector | XPath |
|------|--------------|-------|
| By class | `.product` | `//*[@class="product"]` |
| By ID | `#header` | `//*[@id="header"]` |
| Direct child | `div > p` | `//div/p` |
| Descendant | `div p` | `//div//p` |
| Attribute | `[href]` | `//*[@href]` |
| Contains class | `[class*="prod"]` | `//*[contains(@class, "prod")]` |
| By text | N/A | `//*[text()="Hello"]` |
| Parent | N/A | `//span/parent::div` |
| Following sibling | `h2 + p` | `//h2/following-sibling::p[1]` |

**When to use XPath**:
- ✅ Need to navigate up (parent/ancestor)
- ✅ Need to select by text content
- ✅ Complex attribute conditions
- ✅ Working with XML

**When to use CSS**:
- ✅ Simpler syntax
- ✅ More familiar to web developers
- ✅ Slightly faster in some browsers
- ✅ Working with BeautifulSoup

## Real-World Examples

### E-commerce Product Scraping

```python
from bs4 import BeautifulSoup
import requests

url = 'https://example.com/products'
response = requests.get(url)
soup = BeautifulSoup(response.content, 'lxml')

# Products in grid layout
products = soup.select('div.product-grid > div.product-item')

for product in products:
    # Title (might be in different places)
    title = (
        product.select_one('h2.product-name a') or
        product.select_one('a.product-link') or
        product.select_one('[data-product-title]')
    )
    
    # Price (handle sale prices)
    sale_price = product.select_one('span.sale-price, span.price-sale')
    regular_price = product.select_one('span.regular-price, span.price-regular')
    price = sale_price or regular_price
    
    # Rating (different formats)
    rating = (
        product.select_one('div.rating span[data-rating]') or
        product.select_one('span.stars::attr(class)')  # class="stars-4-5"
    )
    
    # Availability
    in_stock = product.select_one('span.in-stock, button.add-to-cart') is not None
    
    # Image (handle lazy loading)
    img = product.select_one('img.product-image')
    image_url = None
    if img:
        image_url = img.get('src') or img.get('data-src') or img.get('data-lazy')
    
    # Product URL
    link = product.select_one('a.product-link, h2 a')
    product_url = link['href'] if link else None
```

### News Article Scraping

```python
# Article list
articles = soup.select('article.post, div.article-card')

for article in articles:
    # Headline
    headline = article.select_one('h2.headline, h3.title, a.article-title')
    
    # Summary/excerpt
    summary = article.select_one('p.excerpt, div.summary, p.description')
    
    # Author
    author = article.select_one('span.author, a.author-link, [data-author]')
    
    # Date
    date = article.select_one('time, span.date, [datetime]')
    
    # Category/tags
    category = article.select_one('span.category, a.category-link')
    tags = article.select('a.tag, span.tag')
    
    # Article link
    link = article.select_one('a[href*="/article/"], a[href*="/post/"]')
```

### Social Media Post Scraping

```python
# Using Selenium for dynamic content
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get('https://example.com/posts')

# Wait for posts to load
wait = WebDriverWait(driver, 10)
wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, 'div.post')))

# Get posts
posts = driver.find_elements(By.CSS_SELECTOR, 'div.post, article.post-item')

for post in posts:
    # Username
    username = post.find_element(By.CSS_SELECTOR, 'a.username, span.user-name').text
    
    # Post content
    content = post.find_element(By.CSS_SELECTOR, 'div.post-content, p.text').text
    
    # Timestamp
    timestamp = post.find_element(By.CSS_SELECTOR, 'time, span.timestamp').get_attribute('datetime')
    
    # Likes/reactions
    likes = post.find_element(By.CSS_SELECTOR, 'span.like-count, [data-likes]').text
    
    # Comments count
    comments = post.find_element(By.CSS_SELECTOR, 'span.comment-count').text
```

## Debugging Selectors

### Browser DevTools

1. **Chrome/Firefox DevTools**:
   - Right-click element → Inspect
   - In Console: `$$('div.product')` (CSS) or `$x('//div[@class="product"]')` (XPath)
   - Shows matching elements

2. **Test selectors**:
```javascript
// In browser console
document.querySelectorAll('div.product')  // CSS
$x('//div[@class="product"]')             // XPath
```

### Python Debugging

```python
from bs4 import BeautifulSoup

# Test selector interactively
soup = BeautifulSoup(html, 'lxml')

# See if selector matches anything
results = soup.select('div.product')
print(f"Found {len(results)} elements")

# Print HTML of matched elements
for elem in results[:3]:  # First 3
    print(elem.prettify())

# Check if specific element exists
if soup.select_one('#header'):
    print("Header found")
else:
    print("Header not found - check selector")
```

### Common Selector Mistakes

**1. Wrong class selector**:
```python
# ❌ Wrong (treats whole string as class)
soup.select('[class="product featured"]')

# ✅ Correct (elements with both classes)
soup.select('.product.featured')
```

**2. Forgetting direct child vs descendant**:
```python
# Selects all <p> inside div (at any level)
soup.select('div p')

# Selects only direct <p> children
soup.select('div > p')
```

**3. Case sensitivity**:
```python
# HTML classes are case-sensitive
soup.select('.Product')  # Won't match class="product"
soup.select('.product')  # Correct
```

**4. Dynamic class names**:
```python
# ❌ Won't work if class changes (e.g., "product-abc123")
soup.select('.product-abc123')

# ✅ Use partial match
soup.select('[class^="product-"]')  # Starts with "product-"
soup.select('[class*="product"]')   # Contains "product"
```

## Selector Performance Tips

1. **Use specific selectors**:
```python
# Slower (searches entire document)
soup.select('.price')

# Faster (searches within specific container)
container = soup.select_one('#product-list')
prices = container.select('.price')
```

2. **Avoid overly complex selectors**:
```python
# Slower
soup.select('div > ul > li > div > span.price')

# Faster (if unique enough)
soup.select('span.price')
```

3. **Use ID selectors when possible**:
```python
# Fastest (ID is unique)
soup.select_one('#main-content')

# Slower
soup.select_one('div.main-content')
```

4. **Cache frequent selections**:
```python
# If selecting same container multiple times
container = soup.select_one('#product-list')

# Reuse container
for product in container.select('.product'):
    title = product.select_one('.title')
    price = product.select_one('.price')
```
