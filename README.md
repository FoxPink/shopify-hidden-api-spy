# Shopify Store Intelligence Spy

[![Apify Marketplace](https://img.shields.io/badge/Apify-Marketplace-FF7754)](https://apify.com/foxpink/shopify-hidden-api-spy)
[![GitHub](https://img.shields.io/badge/GitHub-Repo-181717)](https://github.com/FoxPink/shopify-hidden-api-spy)

> Extract every product, variant, price, image, and tag from any Shopify store via hidden JSON API. Zero DOM, zero browser — never breaks. Includes competitive intelligence: launch velocity, price range, tag cloud, theme detection, SEO metadata, installed apps, and review analytics at **$0.01/1k results**.

---

## Why This Actor

Most Shopify scrapers use Puppeteer or Playwright to render pages, parse HTML classes, and extract data from DOM elements. The moment a store changes its theme or installs a layout app, those selectors break — costing you time and compute budget.

This Actor takes a different approach: it calls Shopify's **internal JSON API** (`products.json`) — the same endpoint Shopify itself uses to power every storefront on the internet. This API never changes, never breaks, and returns clean structured data regardless of theme, plugins, or design updates.

## Features

- **Zero DOM / Zero Browser** — pure `fetch` calls, no Puppeteer, no headless Chrome, no selectors to break
- **Hidden JSON API** — taps into `https://[store]/products.json?limit=250&page=N` for raw product data
- **Password Protection Detection** — checks `x-shopify-stage` header to detect password-protected stores
- **Bulk Processing** — accepts a single URL or an array of stores for batch intelligence runs
- **Launch Velocity** — counts products published in the last 30 days to measure store activity
- **Price Range Analysis** — calculates min, max, and average variant prices across the entire store
- **Tag Cloud** — aggregates and ranks the top 15 SEO tags used across all products
- **Product Type Breakdown** — identifies the top 5 product categories by volume
- **Variant SKU & Inventory** — per-variant SKU, price, inventory quantity, and availability status
- **Full Product Images** — array of all product images plus main image shortcut
- **Product Descriptions** — raw HTML body for AI pipelines and SEO analysis
- **Collection Mapping** — maps each product to its store collections
- **Store Name Extraction** — pulls store name from homepage `<title>` tag
- **Auto-Retry & Timeout** — built-in retry with exponential backoff for transient failures
- **Theme Detection** — identifies the Shopify theme (Dawn, Debut, etc.) via `Shopify.theme` HTML injection
- **SEO Metadata Extraction** — retrieves meta description, og:title, og:description, og:image from homepage
- **Review Analytics** — extracts aggregate rating and total review count from JSON-LD structured data
- **Installed Apps Detection** — scans homepage for 20+ known Shopify app scripts (Yotpo, Klaviyo, Gorgias, etc.)
- **Store Metadata** — fetches shop name, currency, country, locale via `/meta.json`
- **Proxy Fallback** — transparently falls back to Apify residential proxy on 403/429
- **Tech Stack Detection** — identifies the Shopify theme framework and detects front-end technologies (React, Alpine, Tailwind, etc.) via script analysis.
- **Payment Detection** — detects installed payment gateways (Shopify Payments, PayPal, Stripe, Klarna, etc.) from storefront script tags and checkout hints.

## Use Cases

| Who | Why |
|-----|-----|
| E-commerce Competitor Analysts | Monitor competitor pricing, new product launches, and catalog size |
| Dropshippers | Find winning products by analyzing top-selling Shopify stores |
| Brand Managers | Track brand presence, product mix, and SEO strategy of competitors |
| Market Researchers | Build product catalogs and price databases at scale |
| AI/ML Pipelines | Feed product data into recommendation engines or pricing models |

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `shopifyUrl` | string | — | Single target Shopify store URL (e.g. `gymshark.com`) |
| `shopifyUrls` | array | — | Bulk list of store URLs for batch processing |
| `maxProducts` | integer | `500` | Max products to extract per store (250 per API page) |

Either `shopifyUrl` or `shopifyUrls` must be provided.

### Input Example

```json
{
  "shopifyUrl": "gymshark.com",
  "maxProducts": 200
}
```

Bulk mode:

```json
{
  "shopifyUrls": ["gymshark.com", "allbirds.com", "warbyparker.com"],
  "maxProducts": 500
}
```

## Output

Each product record:

| Field | Type | Example |
|-------|------|---------|
| `id` | integer | `6745123456789` |
| `title` | string | `"Glow Skin Serum V2"` |
| `handle` | string | `"glow-skin-serum-v2"` |
| `vendor` | string | `"Beauty Vendor"` |
| `productType` | string | `"Cosmetics"` |
| `bodyHtml` | string | `"<p>Product description...</p>"` |
| `createdAt` | string | `"2026-05-10T08:30:00Z"` |
| `collections` | array | `["Skincare", "Best Sellers"]` |
| `images` | array | `["https://cdn.shopify.com/img1.jpg", "..."]` |
| `mainImage` | string | `https://cdn.shopify.com/...` |
| `variants` | array | `[{title, sku, price, inventoryQuantity, available}]` |
| `totalVariants` | integer | `3` |
| `minPrice` | number | `29.99` |
| `maxPrice` | number | `39.99` |
| `tags` | array | `["serum", "skincare", "organic"]` |
| `isNewlyLaunched30Days` | boolean | `true` |

A `_summary` entry is appended per store with aggregate intelligence:

| Field | Description |
|-------|-------------|
| `storeName` | Store name extracted from homepage title |
| `totalProducts` | Number of products extracted |
| `totalVariants` | Total variant count across all products |
| `totalCollections` | Number of collections found |
| `globalMinPrice` | Cheapest variant across the store |
| `globalMaxPrice` | Most expensive variant |
| `avgVariantPrice` | Average price across all variants |
| `newlyLaunched30Days` | Products published in the last 30 days |
| `launchVelocity30Days` | Average new products per day (30-day window) |
| `themeName` | Shopify theme name (e.g. Dawn, Debut) |
| `themeRole` | Theme role (main, mobile, etc.) |
| `metaDescription` | Homepage meta description |
| `ogTitle` | Open Graph title tag |
| `ogDescription` | Open Graph description |
| `ogImage` | Open Graph image URL |
| `overallRating` | Aggregate rating from JSON-LD structured data |
| `totalReviews` | Total review count from JSON-LD |
| `appsInstalled` | Detected Shopify apps (Yotpo, Klaviyo, etc.) |
| `currency` | Store currency from `/meta.json` |
| `country` | Store country from `/meta.json` |
| `myshopifyDomain` | MyShopify domain from `/meta.json` |
| `topTags` | Top 15 SEO tags with frequency counts |
| `topProductTypes` | Top 5 product types by volume |

## How It Works

```
Shopify Store → HTTPS test → Detect password protection → products.json pagination → Collection mapping → Product Mapping → Intelligence Summary
```

1. Normalizes the store domain
2. Sends a test request to detect password protection or non-Shopify stores
3. Paginates through `products.json` (250 products per page) with auto-retry until all products or `maxProducts` limit is reached
4. Maps each product to its store collections via `collections.json` and `smart_collections.json`
5. Enriches output with variant details (SKU, inventory, availability), full image array, and HTML body
6. Computes competitive intelligence: launch velocity, price range, tag cloud, and product type stats

## Pricing

**$0.01 per 1,000 results.** One result = one product record. Lightweight fetch-only architecture keeps compute costs near zero. A store with 500 products costs less than a cent to scrape.

## Quick Start

```bash
curl -X POST https://api.apify.com/v2/acts/foxpink~shopify-hidden-api-spy/runs \
  -H "Content-Type: application/json" \
  -d '{
    "shopifyUrl": "gymshark.com",
    "maxProducts": 200
  }' \
  "https://api.apify.com/v2/acts/foxpink~shopify-hidden-api-spy/runs?token=YOUR_API_TOKEN"
```

## Compatibility

- 100% Node.js (18+)
- No browser, no headless, no DOM
- No external scraping libraries — pure `fetch` API
- ESM (ECMAScript Modules)
