# Shopify Store Intelligence Spy

[![Apify Marketplace](https://img.shields.io/badge/Apify-Marketplace-FF7754)](https://apify.com/foxpink/shopify-hidden-api-spy)
[![GitHub](https://img.shields.io/badge/GitHub-Repo-181717)](https://github.com/FoxPink/shopify-hidden-api-spy)

> Extract every product, variant, price, image, and tag from any Shopify store via hidden JSON API. **Sale detection, discount analysis, and collection filtering included.** Zero DOM, zero browser — never breaks.

---

## Why This Actor

Most Shopify scrapers use Puppeteer or Playwright to render pages and parse HTML classes. The moment a store changes its theme, those selectors break.

This Actor calls Shopify's **internal JSON API** (`products.json`) — the same endpoint Shopify itself uses to power every storefront. This API never changes, never breaks, and returns clean structured data regardless of theme or design updates.

## Features

- **Zero DOM / Zero Browser** — pure `fetch` calls, no Puppeteer, no headless Chrome
- **Hidden JSON API** — taps into `products.json?limit=250&page=N`
- **Sale Detection (new)** — extracts `compareAtPrice` (original price), computes `hasSale` and `discountPercent` per product
- **Collection Filter (new)** — optional `collectionUrlFilter` to scope results to a specific collection handle
- **Password Protection Detection** — checks `x-shopify-stage` header
- **Bulk Processing** — single URL or array of stores
- **Launch Velocity** — products published in last 30 days
- **Price Range Analysis** — min, max, avg variant prices
- **Tag Cloud** — top 15 SEO tags
- **Product Type Breakdown** — top 5 categories
- **Variant SKU & Inventory** — per-variant details
- **Collection Mapping** — maps products to collections
- **Tech Stack Detection** — CDN, payment providers, analytics, reviews, live chat, loyalty programs
- **Installed Apps Detection** — 20+ known Shopify apps (Yotpo, Klaviyo, Gorgias, etc.)
- **Review Analytics** — aggregate rating from JSON-LD
- **Proxy Fallback** — transparent fallback to Apify residential proxy on 403/429

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `shopifyUrl` | string | — | Single target store URL |
| `shopifyUrls` | array | — | Bulk list of store URLs |
| `maxProducts` | integer | `500` | Max products per store |
| **`collectionUrlFilter`** (new) | string | — | Filter by collection handle (e.g. `winter-sale`) |

## Output

Each product:

| Field | Type | Example |
|-------|------|---------|
| `title` | string | `"Glow Skin Serum V2"` |
| `minPrice` / `maxPrice` | number | `29.99` / `39.99` |
| **`compareAtPrice`** (new) | number | `49.99` |
| **`hasSale`** (new) | boolean | `true` |
| **`discountPercent`** (new) | number | `25.0` |
| `variants` | array | SKU, price, compareAtPrice, inventory, availability |
| `collections` | array | `["Skincare", "Best Sellers"]` |
| `images` | array | Product image URLs |
| `tags` | array | SEO tags |
| `isNewlyLaunched30Days` | boolean | Launch velocity indicator |

Summary per store includes `productsOnSale`, `avgDiscountPercent`, plus store name, currency, theme, SEO metadata, installed apps, and tech stack.

---

## Pricing

**$0.01 per 1,000 results.** One result = one product record. A store with 500 products costs less than a cent to scrape.

---

## Quick Start

```bash
curl -X POST https://api.apify.com/v2/acts/foxpink~shopify-hidden-api-spy/runs \
  -H "Content-Type: application/json" \
  -d '{"shopifyUrl": "gymshark.com", "maxProducts": 200}' \
  "https://api.apify.com/v2/acts/foxpink~shopify-hidden-api-spy/runs?token=YOUR_API_TOKEN"
```

---

## Why FoxPink?

| vs. Competitor | Their Price | Our Price | Advantage |
|----------------|-------------|-----------|-----------|
| autofacts/shopify | $5/month + usage | **$0.01/1k** | **No subscription, pay-per-use** |
| scrapebase/shopify-store-scraper | $3.99/1k | **$0.01/1k** | **399x cheaper** |
| benthepythondev/shopify-store-scraper | $4.00/1k | **$0.01/1k** | **400x cheaper** |

**Unique features they don't have:** hidden JSON API (zero-DOM, never breaks), `hasSale`/`compareAtPrice`/`discountPercent` analytics, password detection, `collectionUrlFilter`.

---

## FoxPink Studio Ecosystem

Combine with other FoxPink actors for a complete data pipeline:

| Actor | Purpose | Price |
|-------|---------|-------|
| [Email Enricher+](https://apify.com/foxpink/email-enricher-plus) | Email verification & spam trap detection | $0.01/1k |
| [RAG Markdown Chunker](https://apify.com/foxpink/apify-rag-markdown-chunker) | HTML→MD→Chunk→Embed pipeline | $0.01/1k |
| [Odoo Market Intel](https://apify.com/foxpink/odoo-apps-market-intelligence) | Odoo Apps Store scraper & analysis | $0.05/1k |

**Workflow example:** Shopifu → scrape products → Email → enrich customer contacts → analyze pricing trends.

---

## Compatibility

- 100% Node.js (18+)
- No browser, no headless, no DOM
- Pure `fetch` API
