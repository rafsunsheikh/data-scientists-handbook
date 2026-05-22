# Web Scraping

> **TL;DR** — Web scraping extracts data from websites that don't offer APIs. It works by fetching HTML and parsing it, but it's inherently fragile: sites change their structure, content is often JavaScript-rendered, and anti-bot measures actively fight scrapers. The most successful scrapers are polite (respect robots.txt, rate limits, and terms of service), robust (monitor for breaks, alert on schema changes), and reproducible (archive the pages they scrape). Always check for an API first — scraping should be the last resort.

## 1. Scraping approaches

### 1.1 Static HTML

The simplest case: the server returns HTML with the data already embedded.

**Tools:**

| Tool | Speed | Features |
|---|---|---|
| `requests` + `BeautifulSoup` | Moderate | Simple, flexible, Pythonic |
| `requests` + `lxml` | Fast | CSS/XPath selectors, C-based parser |
| `requests` + `selectolax` | Fastest | Modest subset of CSS/XPath, Cython |
| `httpx` + `BeautifulSoup` | Moderate | Async support, HTTP/2 |

**Python:**

```python
import requests
from selectolax.parser import HTMLParser

url = "https://example.com/data-page"
response = requests.get(url, headers={"User-Agent": "DataScienceHandbook (mailto:rafsun.sheikh@audd.digital)"})
response.raise_for_status()

tree = HTMLParser(response.text)
titles = tree.css_text("h2.article-title")
tables = tree.css("table.data-table")
```

**Parsing strategies:**

- **CSS selectors** (`tree.css("div.price")`) — fast, simple, most common.
- **XPath** (`tree.xpath("//div[@class='price']")`) — more powerful, handles axes and predicates.
- **Regex** — last resort. HTML is not regular; regex breaks on edge cases.

### 1.2 Dynamic (JavaScript-rendered) content

Many sites load data via JavaScript after the initial HTML. The data you want isn't in the raw HTML.

**Tools:**

| Tool | Approach | Speed |
|---|---|---|
| **Playwright** | Headless browser (Chromium, Firefox, WebKit) | Moderate |
| **Selenium** | Headless browser (any browser) | Slow |
| **DrissionPage** | Combined page/session control | Fast |
| **curl + reverse-engineer XHR** | Find the API call the JS makes | Fastest (if you can find it) |

**Python:**

```python
from playwright.async_api import async_playwright
import asyncio

async def scrape_dynamic(url):
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        await page.goto(url, wait_until="networkidle")
        # Wait for specific element
        await page.wait_for_selector(".data-loaded")
        data = await page.inner_text(".data-container")
        await browser.close()
        return data

# Better: find the XHR/Fetch API the page calls
# Check Network tab in DevTools → XHR/Fetch requests → JSON response
```

**Pro tip:** Before reaching for a headless browser, check the Network tab in DevTools. Most dynamic sites call a backend API (JSON) that you can call directly — faster, more reliable, no browser overhead.

### 1.3 Infinite scroll / lazy loading

Pages that load more content as you scroll. You need to simulate scrolling.

**Strategies:**

- **Playwright/Selenium:** Scroll to bottom, wait for new content, repeat.
- **Find the pagination API:** Many infinite-scroll pages have a `?page=N` or `?cursor=xxx` API behind the scenes.
- **Scroll count:** Scroll N times, each time waiting for content to load.

```python
# Playwright infinite scroll
page.goto(url)
previous_height = 0
for _ in range(20):  # scroll up to 20 times
    await page.evaluate("window.scrollBy(0, window.innerHeight)")
    await page.wait_for_timeout(2000)  # wait for content
    current_height = await page.evaluate("document.body.scrollHeight")
    if current_height == previous_height:
        break  # no more content
    previous_height = current_height
```

### 1.4 Form submission

Data behind search forms, filters, or login pages.

**Strategies:**

- **Find the POST request:** Use DevTools Network tab to find the form's submission endpoint.
- **Reproduce the POST:** Send the same form data (including hidden fields, CSRF tokens).
- **Session management:** Login once, maintain cookies for subsequent requests.

```python
import requests

session = requests.Session()

# Login
session.post("https://example.com/login", data={
    "username": "user",
    "password": "pass",
    "csrf_token": "abc123",  # from login page
})

# Submit search form
resp = session.post("https://example.com/search", data={
    "query": "data science",
    "category": "all",
    "sort": "relevance",
})
```

## 2. Crawling frameworks

For scraping many pages (not just one), use a crawling framework.

### 2.1 Scrapy

The most mature Python crawling framework.

**Features:**

- **Async architecture:** Twisted-based, handles thousands of concurrent requests.
- **Item pipelines:** Structured data extraction with validation.
- **Middlewares:** User-Agent rotation, proxy rotation, retry logic.
- **Export:** JSON, CSV, XML, Item Dictionary.
- **Sitemap parsing:** Auto-discover URLs from `sitemap.xml`.
- **Robots.txt:** Built-in compliance.
- **Extensions:** Email reporting, logging, throttling.

**Python:**

```python
import scrapy

class DataSpider(scrapy.Spider):
    name = "data_scraper"
    start_urls = ["https://example.com/data-page"]

    def parse(self, response):
        for item in response.css("div.item"):
            yield {
                "title": item.css("h2::text").get(),
                "price": item.css("span.price::text").get(),
                "url": item.css("a::attr(href)").get(),
            }

        next_page = response.css("a.next::attr(href)").get()
        if next_page:
            yield response.follow(next_page, callback=self.parse)
```

**Other frameworks:**

| Framework | Language | Notes |
|---|---|---|
| **Scrapy** | Python | Most mature, largest ecosystem |
| **Colly** | Go | Extremely fast, lightweight |
| **Nutch** | Java | Large-scale, search-engine style |
| **Playwright crawl** | Python/JS | For JS-rendered sites |
| **crawl4ai** | Python | AI-focused scraping with LLM extraction |

## 3. Structured data on pages

Before scraping HTML, check if the page provides structured data.

### 3.1 Schema.org (JSON-LD)

Google, Bing, and other search engines support structured data markup. Often contains the exact data you want in clean JSON.

```python
import json
import re
from selectolax.parser import HTMLParser

def extract_jsonld(html):
    tree = HTMLParser(html)
    scripts = tree.css("script[type='application/ld+json']")
    results = []
    for script in scripts:
        try:
            data = json.loads(script.text())
            results.append(data)
        except json.JSONDecodeError:
            pass
    return results
```

### 3.2 OpenGraph and Twitter Cards

Meta tags for social media sharing — often contain title, description, image, URL.

```html
<meta property="og:title" content="Page Title">
<meta property="og:image" content="https://example.com/image.jpg">
<meta name="twitter:card" content="summary_large_image">
```

### 3.3 RSS / Atom feeds

Many sites publish data feeds. Check for `feed.xml`, `rss.xml`, or `/feed`.

```python
import feedparser
feed = feedparser.parse("https://example.com/feed.xml")
for entry in feed.entries:
    print(entry.title, entry.link, entry.published)
```

## 4. Ethics, legality, and politeness

### 4.1 robots.txt

`https://example.com/robots.txt` tells scrapers which paths are disallowed.

```
User-agent: *
Disallow: /admin/
Allow: /public/
Sitemap: https://example.com/sitemap.xml
```

**Python:**

```python
from urllib.robotparser import RobotFileParser

rp = RobotFileParser()
rp.set_url("https://example.com/robots.txt")
rp.read()
can_scrape = rp.can_fetch("*", "https://example.com/data-page")
```

**Note:** `robots.txt` is a convention, not a law. But ignoring it can lead to IP bans, legal action, and reputational damage.

### 4.2 Terms of Service

Many sites explicitly prohibit scraping in their ToS. Check before scraping.

**Legal risks:**

- **CFAA (US):** Unauthorized access to computer systems (violation of ToS could qualify per *Van Buren* ruling — narrow but still risky).
- **GDPR (EU):** Scraping personal data without legal basis is illegal.
- **Copyright:** Content may be copyrighted; scraping + republishing = infringement.
- **Database Directive (EU):** Sui generis right against extraction of database contents.
- **HiQ vs. LinkedIn:** Scraping publicly available data is more defensible, but non-public data is not.

### 4.3 Politeness guidelines

- **Identify yourself:** Use a descriptive User-Agent with contact info.
- **Rate limit:** 1 request per 1–2 seconds minimum. Respect `Crawl-delay` in robots.txt.
- **Cache results:** Don't re-scrape the same page every minute.
- **Off-peak hours:** Scrape during low-traffic times.
- **Use provided exports:** If the site offers CSV/JSON download, use that instead.
- **Contact the site:** Many sites will share data if you ask.

### 4.4 Rate limiting

```python
import time
import random

def polite_delay(min_delay=1.0, max_delay=3.0):
    """Random delay between requests."""
    time.sleep(random.uniform(min_delay, max_delay))
```

## 5. Anti-bot countermeasures

### 5.1 Common countermeasures

| Measure | What it detects | Bypass methods |
|---|---|---|
| **Cloudflare** | Bots, CAPTCHAs, JS challenge | `cloudscraper`, `crawlee`, residential proxies |
| **DataDome** | Behavioral analysis | Headless browser fingerprint spoofing |
| **PerimeterX** | Browser fingerprinting | Real browser (not headless) |
| **reCAPTCHA** | CAPTCHA challenges | 2Captcha, Anti-Captcha, CapSolver (paid services) |
| **hCaptcha** | CAPTCHA challenges | Same as reCAPTCHA |
| **Akamai Bot Protection** | IP reputation, behavior | Residential proxies, rate limiting |

### 5.2 Fingerprinting

Browsers expose information that distinguishes bots from humans:

- User-Agent string
- Screen resolution
- Installed fonts
- WebGL renderer
- Timezone
- Language settings
- Canvas fingerprint

**Tools for spoofing:**

- `playwright` with `--disable-blink-features=AutomationControlled`
- `undetected-chromedriver` (Selenium)
- `browser-fingerprint` libraries

**Warning:** Circumventing anti-bot measures may violate laws (CFAA, DMCA anti-circumvention, Computer Misuse Act UK). Know the legal risks.

### 5.3 Proxies

| Type | Cost | Anonymity | Speed |
|---|---|---|---|
| **Datacenter** | Cheap | Low (known IP ranges) | Fast |
| **Residential** | Expensive | High (real ISP IPs) | Moderate |
| **Mobile** | Very expensive | Highest (cellular IPs) | Slow |

**Rotation:** Rotate IPs to avoid rate limits and bans. Use proxy providers (Bright Data, Oxylabs, Smartproxy).

## 6. Parsing pitfalls

### 6.1 Encoding

Pages declare encoding in `<meta charset>` or HTTP headers. `requests` guesses `apparent_encoding` (chardet) but may be wrong.

```python
import requests
from chardet import detect

resp = requests.get(url)
# requests guesses, but verify
result = detect(resp.content)
encoding = result["encoding"]  # e.g., "utf-8", "iso-8859-1"
```

### 6.2 JavaScript-rendered content

The HTML you get from `requests` may be a skeleton. The real data loads via JS. Check the raw response vs. "View Page Source" in DevTools.

### 6.3 Anti-scraping HTML

Some sites inject fake data, honeypot links, or obfuscate content:

- **CSS hiding:** `display: none` on bot-visible content.
- **Text color:** White text on white background.
- **Honeypot fields:** Hidden form fields that humans don't fill but bots do.
- **Obfuscated text:** HTML entities, Unicode homoglyphs, zero-width characters.

### 6.4 Pagination traps

- Pages that reset when you visit them out of order.
- Sessions that expire.
- CSRF tokens that change per page.

## 7. Archiving for reproducibility

Scraped data is time-sensitive. The page may change or disappear.

### 7.1 WARC format

Web ARChive format — captures full HTTP requests/responses, headers, and timing.

```python
# pywb for archiving and replaying web captures
from warkapture import capture
with capture("my-capture"):
    requests.get("https://example.com/data-page")
```

### 7.2 Snapshot approach

At minimum, save:

1. The raw HTML (or JSON response).
2. The URL and timestamp.
3. The extracted data.
4. The parsing code (version-controlled).

```python
import json
from datetime import datetime

snapshot = {
    "url": url,
    "timestamp": datetime.utcnow().isoformat(),
    "status_code": resp.status_code,
    "html": resp.text,  # raw HTML
    "extracted": extracted_data,
    "parser_version": "1.2.0",
}
with open(f"snapshots/{slug}.json", "w") as f:
    json.dump(snapshot, f, indent=2)
```

## 8. Maintenance and monitoring

### 8.1 Scrapers break

Sites change their HTML structure, CSS classes, or JavaScript. Monitor for:

- **Schema changes:** Extracted fields are None or empty.
- **Row count changes:** Unexpected drop or spike in results.
- **Error rate:** HTTP errors, timeouts, CAPTCHAs.
- **Data quality:** New values in categorical fields, format changes.

### 8.2 Alerting

```python
def validate_extraction(data, schema):
    """Check that extracted data matches expected schema."""
    for field, expected_type in schema.items():
        if field not in data:
            raise ValueError(f"Missing field: {field}")
        if not isinstance(data[field], expected_type):
            raise TypeError(f"Field {field}: expected {expected_type}, got {type(data[field])}")
```

### 8.3 Version selectors

Store CSS/XPath selectors in a versioned config file, not hardcoded in code. Makes it easy to update without code changes.

```json
{
    "version": "2.1",
    "selectors": {
        "title": "h2.article-title",
        "price": "span.price::text",
        "next_page": "a.next::attr(href)"
    }
}
```

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `requests` | HTTP client for static pages |
| `httpx` | Async HTTP client, HTTP/2 |
| `BeautifulSoup` | HTML/XML parsing |
| `lxml` | Fast HTML/XML parsing with XPath |
| `selectolax` | Fastest CSS selector parsing |
| `Scrapy` | Full crawling framework |
| `Playwright` | Headless browser, dynamic content |
| `Selenium` | Headless browser (older) |
| `crawl4ai` | AI-powered web extraction |
| `pdfplumber` | PDF table extraction (from scraped URLs) |
| `feedparser` | RSS/Atom feed parsing |
| `chardet` | Encoding detection |
| `python-whois` | Domain registration info |
| `warcio` | WARC file reading/writing |
| `cloudscraper` | Cloudflare bypass |

## 10. References

- Lutz, M. *Learning the `requests` Library*. O'Reilly.
- Scrapy Documentation — https://docs.scrapy.org/
- Playwright Documentation — https://playwright.dev/python/
- W3C Robots Exclusion Protocol — https://www.robotstxt.org/
- Google Search Central — https://developers.google.com/search/docs/crawling-indexing/robots/intro
- *Web Scraping with Python* (O'Reilly) — K. Hyde.
- HiQ Labs, Inc. v. LinkedIn Corp. (2019) — legal precedent on public data scraping.
