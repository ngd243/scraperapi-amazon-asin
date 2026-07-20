# Web Scraping API for Python: Is ScraperAPI the Best Tool for Developers? How Does It Work, What Plans Are Available, and Which One Should You Actually Buy?

If you've spent any time building a scraper in Python, you already know the drill. You write a clean `requests.get()` call, everything works fine on your laptop, you push it to production — and within 48 hours, half your requests are getting blocked, CAPTCHAs are everywhere, and your IP address is on some list you'll never get off of. The actual scraping logic? Twenty lines. The proxy rotation, header management, ban detection, and retry logic? Three weeks of your life.

That's the real reason a web scraping API for Python exists. Not because scraping is hard — it isn't — but because the *infrastructure* around scraping at any meaningful scale is genuinely painful to maintain.

ScraperAPI is one of the most widely-used solutions in this space, and for good reason. Let's dig into what it actually does, whether the pricing makes sense for your use case, and how to get it running in Python in about five minutes.

---

**What Is ScraperAPI, and Why Do Python Developers Use It?**

ScraperAPI, founded in 2018 by Daniel Ni (a Yale grad and ex-Wall Street developer), hides all the messy infrastructure of large-scale scraping behind a single HTTP endpoint. You pass it a URL — with a few optional parameters — and it hands you back the HTML (or parsed JSON for supported sites). Everything else — proxy rotation across 40 million+ IPs in 50+ countries, CAPTCHA solving, JavaScript rendering, automatic retries, and anti-bot bypass — happens server-side.

For Python developers, the integration is literally a URL swap. Instead of calling the target site directly, you point your `requests.get()` at ScraperAPI's endpoint. That's it. The API is stateless, there's no SDK required (though one exists), and the documentation is genuinely good.

The company now serves over 10,000 brands — including Deloitte, Sony, and Alibaba — and processes around 36 billion API requests per month. In April 2026, they also acquired Traject Data (the company behind Rainforest API and SerpWow), expanding their catalog to include structured SERP and e-commerce data APIs. It's grown from a "proxy rotator with a nice interface" into a more complete managed data-collection platform.

> **The quick pitch**: If you're tired of maintaining your own proxy pools and want to ship scraping pipelines faster, ScraperAPI is a very reasonable place to start.

---

**How to Use ScraperAPI in Python: The 5-Minute Setup**

Let's get the practical stuff out of the way first, because this is genuinely where ScraperAPI earns its reputation for developer experience.

**Method 1: The Direct API Endpoint (Simplest)**

python
import requests
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'
url = 'https://example.com/products'

params = {'api_key': API_KEY, 'url': url}
response = requests.get('http://api.scraperapi.com/', params=urlencode(params))
print(response.text)


That's genuinely it for a basic request. You don't need to manage proxies, rotate user agents, or handle bans. ScraperAPI does it automatically.

**Method 2: With Retry Logic and BeautifulSoup Parsing**

For production use, you'll want to handle the occasional failed request (ScraperAPI returns a 500 for ones it can't complete). Here's a more robust pattern:

python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'
NUM_RETRIES = 3
list_of_urls = ['https://quotes.toscrape.com/page/1/', 'https://quotes.toscrape.com/page/2/']

scraped_data = []

for url in list_of_urls:
    params = {'api_key': API_KEY, 'url': url}
    for _ in range(NUM_RETRIES):
        try:
            response = requests.get('http://api.scraperapi.com/', params=urlencode(params))
            if response.status_code in [200, 404]:
                break
        except requests.exceptions.ConnectionError:
            response = None
    
    if response and response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        quotes = soup.find_all('div', class_='quote')
        for block in quotes:
            scraped_data.append({
                'quote': block.find('span', class_='text').text,
                'author': block.find('small', class_='author').text
            })

print(scraped_data)


**Method 3: Concurrent Threads for High-Volume Scraping**

When you need to scrape thousands of pages, you scale by adding threads — matching the concurrent thread limit of your plan:

python
import requests
import concurrent.futures
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'
NUM_THREADS = 5
NUM_RETRIES = 3

list_of_urls = ['https://quotes.toscrape.com/page/1/', 'https://quotes.toscrape.com/page/2/']
results = []

def scrape_url(url):
    params = {'api_key': API_KEY, 'url': url}
    for _ in range(NUM_RETRIES):
        try:
            response = requests.get('http://api.scraperapi.com/', params=urlencode(params))
            if response.status_code in [200, 404]:
                return response.text
        except requests.exceptions.ConnectionError:
            pass
    return None

with concurrent.futures.ThreadPoolExecutor(max_workers=NUM_THREADS) as executor:
    results = list(executor.map(scrape_url, list_of_urls))


A few things Python developers should know about timeouts and SSL:
- **Don't set a timeout below 60 seconds** — ScraperAPI automatically retries difficult requests for up to 60 seconds before returning a 500.
- **In proxy mode, set `verify=False`** — SSL certificate verification needs to be disabled when routing through the proxy port.
- ScraperAPI charges only for successful responses (HTTP 200 and 404) — 500 errors don't consume credits.

👉 [Start scraping with ScraperAPI's free tier — no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

**The Credit System: The One Thing You Must Understand Before Buying**

Here's where most reviews let Python developers down. ScraperAPI sells plans with numbers like "100,000 API credits" on the Hobby plan, and "1,000,000 credits" on Startup. That sounds generous. The problem is that one credit does *not* always equal one request.

The actual cost per request depends on the domain you're scraping and the parameters you enable. Here's the table that should be front and center on every ScraperAPI review:

**Domain-based credit multipliers (automatic — you don't opt in):**

| Domain Type | Credits per Request | Examples |
|---|---|---|
| Standard web pages | 1 | Blogs, news, general HTML |
| E-commerce | 5 | Amazon, eBay, Walmart |
| SERP | 25 | Google, Bing (all subdomains) |
| Social media | 30 | LinkedIn |

**Parameter-based extra costs (only when you enable them):**

| Parameter | Extra Credits |
|---|---|
| `render=true` (JavaScript rendering) | +10 |
| `premium=true` (premium proxies) | +10 |
| `screenshot=true` | +10 |
| `ultra_premium=true` | +30 |
| `premium=true` + `render=true` combined | +25 (NOT +20) |
| `ultra_premium=true` + `render=true` combined | +75 (NOT +40) |
| Cloudflare/Datadome/PerimeterX bypass | +10 each |

Notice the combined-parameter rows. Enabling premium proxy AND JavaScript rendering doesn't cost $$10 + 10 = 20$$ extra credits — it costs +25. Ultra-premium plus rendering isn't $$30 + 10 = 40$$ — it's +75. These non-linear combinations are documented, but you have to dig for them.

**The real-world math**: On the Hobby plan (100,000 credits), if you're scraping Amazon product pages with JavaScript rendering enabled, each request costs $$5 + 10 = 15$$ credits. Your "100,000 credit" plan delivers about **6,666 actual Amazon pages**, not 100,000.

Parameters that cost *zero* extra credits: `country_code`, `wait_for_selector`, `session_number`, `device_type`, `output_format`, `keep_headers`, `autoparse`.

---

**ScraperAPI Pricing Plans: Every Tier Explained**

Here's a complete breakdown of all seven plans currently available, including annual pricing (10% off on all paid plans):

| Plan | Monthly Price | Annual (per mo) | API Credits | Concurrent Threads | Geotargeting | Pay-As-You-Go | Purchase |
|---|---|---|---|---|---|---|---|
| **Free** | $0 | — | 1,000/mo | 5 | No | No |  [Get Free Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49 | $44.10 | 100,000 | 20 | US & EU only | No |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149 | $134.10 | 1,000,000 | 50 | US & EU only | No |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299 | $269.10 | 3,000,000 | 100 | Global (50+ countries) | No |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** | $475 | $427.50 | 5,000,000 | 200 | Global | Yes |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** | $975 | $877.50 | 10,500,000 | 300 | Global | Yes |  [Get Professional Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** | $1,975 | $1,777.50 | 21,500,000 | 500 | Global | Yes |  [Get Advanced Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22,000,000+ | 500+ | Global + dedicated | Yes |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

**A few things worth flagging:**

- **Credits do not roll over.** Unused credits expire at the end of the billing cycle.
- **Pay-As-You-Go is only available from Scaling and up.** On Hobby, Startup, and Business, if you exhaust your credits mid-month, you're cut off until renewal or upgrade.
- **Global geotargeting (beyond US & EU) requires the Business plan or higher.** On Hobby and Startup you can only target US and EU.
- **7-day free trial** gives you 5,000 credits to test at a realistic scale before committing to a paid plan.
- **7-day no-questions-asked refund** policy applies to all paid plans.

👉 [Compare all ScraperAPI plans and start your free trial](https://www.scraperapi.com/?fp_ref=coupons)

---

**Which Plan Should You Actually Choose?**

This is where most people overthink it. The honest answer depends almost entirely on what you're scraping, at what volume, and whether you need JavaScript rendering.

**If you're just getting started:** Start with the free tier (1,000 credits/month, 5 concurrent threads). It's genuinely free, no credit card required. Run your actual target URLs through it. Find out which ones need JS rendering or hit a domain multiplier. Then calculate your real monthly credit consumption *before* picking a paid plan.

**If you're a solo developer or freelancer:** The **Hobby plan at $49/month** covers 100,000 credits and 20 concurrent threads — that's plenty for side projects and client work on simpler HTML sites. Just be aware that Amazon or Google scraping will eat credits 5–25x faster than standard pages.

**If you're running a startup or small team:** The **Startup plan at $149/month** (1M credits, 50 threads) or **Business plan at $299/month** (3M credits, 100 threads, global geotargeting) are the realistic starting points for production pipelines. Business is the first tier where you get country-level geotargeting across 50+ countries — worth it if you need data from specific regions.

**If you're scraping at serious scale:** The **Scaling plan at $475/month** is labeled "most popular" for a reason — 5M credits, 200 threads, and crucially, it's the first tier where Pay-As-You-Go kicks in. This means if you exceed your monthly allocation, you continue scraping at a fixed per-credit rate rather than getting hard-blocked. For production systems where uptime matters, this is a meaningful difference.

**If your pipeline runs 24/7:** Look at **Professional ($975/month)** or **Advanced ($1,975/month)**. These tiers include priority support and significantly higher thread limits (300 and 500 respectively), which matters more than raw credits when you're running concurrent jobs continuously.

---

**Where ScraperAPI Performs Well — and Where It Doesn't**

Not all websites are created equal, and neither is ScraperAPI's performance on them. Based on independent benchmarks from Scrapeway (April 2026):

**Sites where ScraperAPI is strong:**

| Target | Success Rate | Notes |
|---|---|---|
| Zillow | ~100% | Excellent real estate scraping |
| Amazon | ~98% | Top-tier with structured data endpoints |
| Etsy | ~99% | Consistent e-commerce performance |
| LinkedIn | ~95% | Works, but costs 30 credits/request |
| Walmart | ~93% | Reliable e-commerce data |
| Google SERP | Strong | Structured data endpoint included |

**Sites where ScraperAPI struggles:**

| Target | Success Rate | Notes |
|---|---|---|
| Instagram | ~0% | Complete failure in benchmarks |
| Twitter/X | ~0% | Not supported |
| Booking.com | ~0% | Anti-bot defeats it consistently |
| Realtor.com | ~12% | Poor performance |

The key insight here: ScraperAPI is excellent for the most commercially valuable scraping targets — Amazon, Google, Walmart, real estate. For social media platforms or travel booking sites, it's not the right tool at all.

There's one other limitation worth mentioning explicitly: **ScraperAPI forbids scraping data behind login walls.** It supports session persistence via the `session_number` parameter, but it cannot handle form submission, two-factor authentication, or any flow that requires an actual authenticated browser session. If your use case involves scraping logged-in content, you'll need a different approach.

---

**ScraperAPI's Structured Data Endpoints: Skipping the HTML Parsing Step**

One of ScraperAPI's genuinely useful features for Python developers is its set of 18 structured data endpoints (SDEs). Instead of getting raw HTML back and writing your own BeautifulSoup/lxml parsing logic, these endpoints return clean, parsed JSON directly.

The supported platforms include:

- **Amazon**: Product details (by ASIN), search results, competitor offers — 21 regional marketplaces supported, 18+ data fields per product
- **Google**: SERP results, Shopping, Maps, News, Jobs
- **Walmart**: Product, Search, Category, Reviews
- **eBay**: Product, Search
- **Redfin**: Search, Agent Details, Rental Properties, For Sale listings

For Python developers, this is a significant time saver. Rather than building and maintaining a parser for Amazon product pages (which Amazon changes frequently), you hit the SDE endpoint and get a clean dictionary back.

python
import requests
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'

# Get Amazon product data by ASIN
params = {
    'api_key': API_KEY,
    'asin': 'B08N5WRWNW',
    'country': 'us'
}
response = requests.get('https://api.scraperapi.com/structured/amazon/product', 
                        params=urlencode(params))
product_data = response.json()
print(product_data['name'], product_data['price'])


The cost? Amazon SDEs consume 5 credits per request — same as a standard Amazon page request. For the development time saved on parser maintenance, that's reasonable.

SDEs are available on all plans, including the free tier.

---

**What Real Users Are Saying**

ScraperAPI holds a **4.5/5 on Trustpilot** (43 reviews), **4.4/5 on G2** (16 reviews), and **4.6/5 on Capterra** (62 reviews). The Capterra sub-ratings tell an interesting story: Ease of Use scores 4.9/5, suggesting the developer experience genuinely lives up to the marketing.

The consistent praise across platforms covers the same themes: quick setup, solid documentation, reliable performance on mainstream scraping targets, and responsive customer support.

The consistent criticisms are equally consistent: the credit multiplier system catches first-time users off guard, credits don't roll over, and performance on harder or less common targets can be unreliable. One pattern in Reddit threads involves users expecting their headline credit count to translate directly to request count, then being surprised mid-month.

The lesson isn't that ScraperAPI is deceptive — it isn't. The multiplier system is documented. The lesson is to test your specific targets before committing to a plan.

---

**Frequently Asked Questions**

**Does ScraperAPI have a free plan?**
Yes. The free plan gives you 1,000 credits per month with up to 5 concurrent connections, permanently — no time limit. When you first sign up, you also get a 7-day trial period with 5,000 credits to test at a more realistic scale. 👉 [Sign up for free here](https://www.scraperapi.com/?fp_ref=coupons)

**Does ScraperAPI work with Python?**
Absolutely — it's one of the most Python-friendly scraping APIs around. The integration is a standard `requests.get()` call with your API key and target URL as parameters. There's also an official Python SDK for those who prefer it.

**Can I use ScraperAPI to scrape JavaScript-heavy sites?**
Yes, by adding `render=true` to your request parameters. This runs the page through a headless browser server-side and returns the fully rendered HTML. It costs an extra 10 credits per request on top of the base domain cost.

**What happens when I run out of credits?**
On Scaling, Professional, Advanced, and Enterprise plans, a Pay-As-You-Go option activates automatically so you can continue scraping at a fixed per-credit rate. On Hobby, Startup, and Business plans, scraping stops until your plan renews or you upgrade.

**Is there a discount for annual billing?**
Yes — all paid plans are 10% cheaper when billed annually.

---

**Final Thoughts**

For Python developers, the search for a reliable web scraping API basically bottoms out at a few key questions: Does it handle the sites I actually need to scrape? Is the integration straightforward? And does the pricing model make sense for my volume?

ScraperAPI answers "yes" to all three for the most common use cases. The Python integration is genuinely dead simple. The infrastructure it manages — proxy rotation, CAPTCHA handling, retry logic — would take weeks to build and maintain yourself. And for high-value targets like Amazon, Google, and real estate platforms, the success rates are competitive.

The credit multiplier system requires a bit of upfront math, but it's not a trap — it's a reasonable way to price more expensive requests at a higher rate. The key habit to build is checking your real credits-per-request on your specific targets before scaling up.

If you're evaluating it for a new scraping project in Python, the free tier is a genuinely useful starting point. No credit card, no time limit. Plug in your actual target URLs, see what the credit consumption looks like, then make a buying decision based on real data rather than headline numbers.

👉 [Try ScraperAPI for free — 1,000 credits/month, no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)
