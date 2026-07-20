# ScraperAPI MCP Server: The Complete Setup Guide — How to Connect Claude, Cursor & Windsurf to Live Web Data, Which Plan Is Actually Worth It, and What Nobody Tells You About Credit Costs (With Full Pricing Breakdown)

So you heard about the ScraperAPI MCP server and now you're wondering: does this thing actually work, how do I set it up, and which plan should I be on? Let's walk through it — properly, without the fluff.

---

## **What Is the ScraperAPI MCP Server, and Why Should You Care?**

Here's the short version: you're building something with an AI — Claude, Cursor, your own agent loop — and at some point it needs to go look at a real webpage. Maybe pull product prices, check search results, grab article content. The problem is that AI models can't browse the web on their own, and wiring up a custom scraping solution to every agent you build is genuinely tedious.

That's exactly the gap the ScraperAPI MCP server plugs.

MCP stands for Model Context Protocol — it's an open standard that Anthropic introduced to let AI clients talk directly to external data tools using a consistent interface. Instead of writing custom integration code for every service, you declare an MCP server in your client config, and your AI can call its tools as naturally as it uses any other capability.

The ScraperAPI MCP server turns your LLM client into something that can scrape any public webpage, run Google searches, pull Amazon product data, crawl entire domains, and even parse structured data from Walmart, eBay, and Redfin — all with a single plain-language prompt. No manual proxy management. No headless browser setup. No CAPTCHA-solving boilerplate.

👉 [Try ScraperAPI free — 5,000 trial credits, no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Two Modes: Remote vs. Local — Which One Should You Use?**

The ScraperAPI MCP server ships in two flavors, and the right one depends on what you're trying to do.

**Remote (hosted) server** — This requires zero local setup. ScraperAPI runs the server on their infrastructure; you just point your client at it. It starts automatically when your AI client needs it, and you don't need Python or Docker installed. If you're using Claude Desktop or just want things working in ten minutes, this is the one.

**Local (self-hosted) server** — You run the server yourself, via Python or Docker. It gives you more control (custom rate limits, timeout settings, image size caps) and works in fully offline or private environments. You'll need Python 3.11+ or Docker, but the actual install is a single `pip install` command.

For most developers, the local server is the better choice once you're past initial testing — it's more configurable, and the setup really does take about two minutes.

---

## **How to Set Up the ScraperAPI MCP Server (Step by Step)**

### **Prerequisites**

- A ScraperAPI account and API key — you can get one with a 👉 [free 7-day trial here](https://www.scraperapi.com/?fp_ref=coupons)
- Python 3.11+ (for the Python install path) **or** Docker
- Your AI client of choice: Claude Desktop, Claude Code, Cursor, Windsurf, Cline, or VSCode with GitHub Copilot

### **Step 1: Install the MCP Server Package**

bash
pip install scraperapi-mcp-server


That's literally it for installation.

### **Step 2: Configure Your Client**

Add the following JSON block to your client's MCP configuration file, replacing `<YOUR_SCRAPERAPI_API_KEY>` with your actual key:

json
{
  "mcpServers": {
    "ScraperAPI": {
      "command": "python",
      "args": ["-m", "scraperapi_mcp_server"],
      "env": {
        "API_KEY": "<YOUR_SCRAPERAPI_API_KEY>"
      }
    }
  }
}


**If you're using Docker instead:**

json
{
  "mcpServers": {
    "ScraperAPI": {
      "command": "docker",
      "args": [
        "run", "-i", "-e",
        "API_KEY=${API_KEY}",
        "--rm",
        "scraperapi-mcp-server"
      ]
    }
  }
}


### **Step 3: Client-Specific Setup Paths**

| Client | Where to Add the Config |
|---|---|
| **Claude Desktop** | Settings → Developer tab → Edit Config |
| **Claude Code** | `.claude/settings.json`, or run: `claude mcp add scraperapi -e API_KEY=... -- python -m scraperapi_mcp_server` |
| **Cursor** | Settings → Tools & Integrations → Add MCP Server → Manual |
| **Windsurf** | Settings → Cascade settings → MCP server → gear icon → `mcp_config.json` |
| **Cline (VS Code)** | Cline panel → MCP Servers icon → Configure tab → `cline_mcp_settings.json` |

Once the config is in place, your client launches the server automatically whenever you need it.

### **Step 4: Optional Environment Variables**

The MCP server has a few tunable settings via environment variables:

| Variable | Default | What It Does |
|---|---|---|
| `API_KEY` | *(required)* | Your ScraperAPI API key |
| `API_TIMEOUT_SECONDS` | `70` | Per-request timeout |
| `RATE_LIMIT_MAX_CALLS` | `10` | Max tool calls per rate-limit window |
| `RATE_LIMIT_WINDOW_SECONDS` | `60` | Length of the rate-limit window |
| `IMAGE_SIZE_LIMIT_BYTES` | `700000` | Max inline image size |

---

## **What Can the ScraperAPI MCP Server Actually Do?**

This is where it gets interesting. The server isn't just a "fetch this URL" wrapper — it exposes a full toolkit of capabilities that your AI can call from natural language.

### **Core Scraping Tool: `scrape`**

The foundation. Send any public URL, get back the page content. Optional parameters let you:

- Enable **JavaScript rendering** (`render=true`) for React/Vue/Angular apps — costs 10 extra credits per request
- Target a specific **country** (`country_code`) for geo-restricted content
- Use **premium proxies** (`premium=true`) for sites with moderate anti-bot protection
- Activate **ultra-premium mode** (`ultra_premium=true`) for the heaviest Cloudflare and DataDome-protected sites
- Choose output format: `markdown` (default), `text`, `csv`, or `json`
- Auto-parse supported sites into structured data with `autoparse=true`

Example prompt you'd actually type to your AI:
> *"Scrape this URL and extract all product names and prices. If the page uses JavaScript, enable rendering."*

### **Structured Data Endpoints (SDEs)**

These skip the raw HTML entirely and return clean, pre-parsed JSON. Available for:

- **Google** — Search results, News, Jobs, Shopping, Maps
- **Amazon** — Product details by ASIN, search results, seller offers
- **Walmart** — Product search, product details, category listings, customer reviews
- **eBay** — Product search and listing details
- **Redfin** — For-sale listings, rentals, search results, agent profiles

Each endpoint accepts `country_code`, `tld`, and `output_format` parameters. For teams that need structured e-commerce data, these alone can justify the subscription.

### **Async Crawler**

Three tools working together for domain-wide crawls:

- `crawler_job_start` — launch a crawl from a starting URL, with regex filters for which links to follow, max depth, or a credit budget cap
- `crawler_job_status` — poll until the job finishes
- `crawler_job_delete` — cancel a running crawl

Crawl jobs also support scheduled recurrence (hourly, daily, weekly, monthly) and webhook callbacks so results land wherever you need them.

### **AI Parser**

Build a reusable extraction template from a few example URLs, then apply it to any similar page structure without rewriting prompts. The parser runs as a two-phase background job: create it, wait for it to finish generating, then point it at any URL that matches the template. One credit per parse call, and it learns from the examples you provide.

---

## **The March 2026 MCP Update: What Changed**

ScraperAPI significantly expanded the MCP server's capabilities in its March 2026 release. A few highlights worth knowing:

- **All Structured Data Endpoints** (Google, Amazon, Walmart, eBay, Redfin) are now callable directly via MCP — previously these required separate API calls outside the MCP flow
- **Full crawler integration** was added, meaning your agent can now autonomously kick off a multi-page crawl without leaving the MCP context
- **VSCode + GitHub Copilot** gained official MCP support, joining Claude, Cursor, Windsurf, and Cline
- A new **ScraperAPI MCP Skill** was added — installable via `npx skills add scraperapi/scraperapi-skills` — which gives your agent intelligent guidance about which tool to call for which task, auto-detects the right approach for scraping vs. search vs. crawl requests, and advises on cost optimization and pagination

To get all the latest capabilities, just upgrade: `pip install --upgrade scraperapi-mcp-server`

---

## **Understanding ScraperAPI Credits: The Part That Actually Matters**

This is the thing that surprises people most. Your plan comes with a monthly credit budget, but not every request costs the same number of credits.

**Base credit costs by domain type:**

| Domain Type | Credits per Request |
|---|---|
| Standard pages (blogs, docs, etc.) | 1 credit |
| Amazon | 5 credits |
| Google / Bing (all subdomains) | 25 credits |
| LinkedIn | 30 credits |

**Parameter add-ons:**

| Parameter | Extra Credits |
|---|---|
| `render=true` (JS rendering) | +10 credits |
| `premium=true` | +10 credits |
| `screenshot=true` | +10 credits |
| `ultra_premium=true` | +30 credits |
| `premium=true` + `render=true` together | 25 credits total |
| `ultra_premium=true` + `render=true` together | 75 credits total |

**Anti-bot bypass surcharges:**

| Protection System | Extra Credits |
|---|---|
| Cloudflare bypass | +10 credits |
| Cloudflare Turnstile bypass | +10 credits |
| DataDome bypass | +10 credits |
| PerimeterX / Human bypass | +10 credits |

The important detail: **you're only charged for successful responses** (HTTP 200 or 404). If ScraperAPI fails to fetch the page, you don't burn credits.

Before you commit to a plan, use the **Domain Multiplier tool** in your ScraperAPI dashboard to check the actual per-request cost for your specific target URLs. A 100,000-credit Hobby plan genuinely covers 100,000 plain blog requests — but only about 6,600 Amazon scrapes with JS rendering enabled. The gap matters.

> **Quick math example**: Scraping 50,000 Amazon product pages per month with rendering on = 50,000 × (5 + 10) = 750,000 credits needed. That puts you firmly in Startup or Business territory.

---

## **ScraperAPI Plans: Full Comparison Table**

All plans include: JS rendering, premium proxies, JSON auto-parsing, rotating proxy pools, custom headers, CAPTCHA/anti-bot bypass, custom sessions, automatic retries, unlimited bandwidth, and a 99.9% uptime SLA.

| Plan | Monthly Price | Annual Price (10% off) | API Credits/Month | Concurrent Threads | Geotargeting | Pay-as-you-go | Action |
|---|---|---|---|---|---|---|---|
| **Free Trial** | $0 (7 days) | — | 5,000 (one-time) | 5 | Limited | ❌ |  [Start Free Trial](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only | ❌ |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only | ❌ |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global | ❌ |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** ⭐ Most Popular | $475/mo | $427.50/mo | 5,000,000 | 200 | Global | ✅ |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** | $975/mo | $877.50/mo | 10,500,000 | 300 | Global | ✅ |  [Get Professional Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** | $1,975/mo | $1,777.50/mo | 21,500,000 | 500 | Global | ✅ |  [Get Advanced Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global | ✅ |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

**A few things the table doesn't show:**

- **Analytics history**: Hobby and Startup are capped at 30 days of dashboard history. Business and above get unlimited history.
- **Pay-as-you-go overflow** only kicks in at Scaling and above. On lower tiers, hitting your credit cap means manual intervention.
- **Credits don't roll over** — unused credits reset at each renewal date, so over-buying doesn't help you.
- Annual billing saves a flat 10% — applied automatically at checkout, no coupon code needed.

---

## **Which Plan Should You Actually Choose?**

**Free Trial** — Always start here. No credit card, 5,000 credits, seven days. Point it at your real target URLs, check the credit cost in the dashboard, and size your plan from actual data rather than guesswork.

**Hobby ($49/mo)** — Personal projects, side hustles, prototypes. If you're scraping plain content pages without heavy anti-bot protection, 100,000 credits goes a long way. Good for initial MCP experiments where you're figuring out your actual usage patterns.

**Startup ($149/mo)** — You've graduated from casual scraping. A small SaaS product, a freelance data workflow, an agency running jobs for a handful of clients. The jump to 1,000,000 credits and 50 concurrent threads is meaningful, and you're still under $150/month.

**Business ($299/mo)** — The inflection point. This is where global geotargeting unlocks (Hobby and Startup are US/EU only), analytics history goes unlimited, and you get 3,000,000 credits with 100 concurrent threads. If you're running production workloads that other parts of your product depend on, this is probably the floor.

**Scaling ($475/mo)** — ScraperAPI calls this their most popular plan for a reason. Pay-as-you-go overflow means you're never hard-stopped mid-month, and 5 million credits with 200 threads handles serious volume. Good for teams that have grown past Business but aren't at the agency-infrastructure scale yet.

**Professional / Advanced / Enterprise** — At this point you're past "which plan" and into "how do we keep this predictable and scalable." Priority support kicks in, pay-as-you-go is standard, and the credit pools scale into the tens of millions. If you're at this volume, talking directly to their sales team is worth the time.

---

## **What Developers Actually Say About ScraperAPI**

The pattern across Trustpilot (around 4.5/5) and G2 (around 4.4/5) is consistent: clean documentation, genuinely simple integration, responsive support. Developers who come from managing their own proxy infrastructure repeatedly note that the drop-in simplicity is the main value prop — you replace your proxy library with one API call, and the infrastructure headaches disappear.

The most common friction point isn't reliability — it's the credit multiplier system being more complex than the headline number suggests. People buy 100,000 credits, start scraping Amazon with rendering, and are confused when the balance drops fast. The dashboard's Domain Multiplier tool addresses this, but it requires you to know to look for it.

Performance is also target-dependent. ScraperAPI consistently handles mainstream sites (Amazon, standard e-commerce, GitHub, news sites) very well. Sites with frequently-rotating, aggressive anti-bot systems (certain fintech platforms, heavy Cloudflare Enterprise deployments) can be more variable.

---

## **Practical Prompts to Get Started With the MCP Server**

Once your server is configured, here are some prompts you can literally paste into Claude, Cursor, or whichever client you're using:

> *"Please scrape this URL: [URL]. If you receive a 500 server error, identify the website's geo-targeting and add the corresponding `country_code`. If errors continue, upgrade to `premium=true`. For persistent failures, activate `ultra_premium=true`."*

> *"Can you scrape [URL] to extract [specific data]? If the request returns missing or incomplete data, set `render=true` to enable JS rendering."*

> *"Search Google for [query] and return the top 10 organic results with titles, URLs, and snippets."*

> *"Get the product details for Amazon ASIN B08N5WRWNW in JSON format."*

> *"Start a crawl of [domain] following all links matching the pattern [regex], with a maximum depth of 3. Report back when done."*

---

## **The Bottom Line**

The ScraperAPI MCP server is one of those tools that solves a genuinely annoying problem in a clean way. If you're building AI agents that need real-time web data — and increasingly, most useful agents do — wiring up MCP instead of bespoke API calls is just the smarter approach. One config file, and your AI can scrape, search, crawl, and extract structured data from a long list of platforms, all from natural language.

The setup is fast. The tool coverage is wide. And the credit system, once you understand the multipliers, is actually pretty fair — you pay for successful results, not attempts.

Start with the free trial. Run your actual targets through it. Check the credit costs. Then pick the plan that matches your real usage, not the one that sounds like enough on paper.

👉 [Get started with 5,000 free trial credits — no credit card required](https://www.scraperapi.com/?fp_ref=coupons)
