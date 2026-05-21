# Retail and E-commerce

> **TL;DR** — Retail/e-commerce data centers on a four-way join: **customer × product × time × channel**. Most of the work is keeping those four dimensions clean: identity stitching across devices and sessions, SKU hierarchies that change weekly, transactional vs. behavioral time, and online vs. in-store vs. marketplace channels. Without that, every metric — conversion, AOV, LTV, attribution — silently disagrees across teams.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Tabular transactional** | Orders, line items, payments, refunds, fulfillments |
| **Event / clickstream** | Page views, product views, add-to-cart, search, scroll, hover |
| **Catalog (hierarchical categorical)** | SKUs → variants → products → subcategory → category → department |
| **Inventory time series** | Stock by SKU × location, snapshots and movements |
| **Text** | Search queries, product titles & descriptions, reviews, support tickets |
| **Image** | Product photos, user-uploaded photos, review images |
| **Geospatial** | Shipping addresses, store locations, delivery zones |
| **Time series** | Sales by SKU / day / store, demand forecasting |
| **Graph** | Co-purchase, co-view, recommendation graph |

## 2. Common sources

| Layer | Source |
|---|---|
| **E-commerce platform** | Shopify, BigCommerce, Magento / Adobe Commerce, commercetools, WooCommerce, Salesforce Commerce Cloud |
| **POS / store** | Square, Lightspeed, Toast (food), Oracle Retail, NCR |
| **OMS / WMS / ERP** | Manhattan, Blue Yonder, NetSuite, SAP, Microsoft Dynamics |
| **Web/app analytics** | GA4, Adobe Analytics, Snowplow, Heap, Mixpanel |
| **CDP / event router** | Segment, mParticle, RudderStack |
| **Marketing platforms** | Klaviyo, Braze, Iterable; Meta Ads, Google Ads, TikTok Ads |
| **Marketplaces** | Amazon (SP-API), eBay, Etsy, Walmart, Flipkart, Lazada |
| **Reviews** | Yotpo, Bazaarvoice, Trustpilot, native platform |
| **Fulfillment / shipping** | ShipStation, ShipBob, EasyPost, AfterShip; carrier APIs (UPS, FedEx, USPS) |
| **Payment** | Stripe, Adyen, PayPal, Braintree, Klarna, Affirm |
| **Loyalty / CRM** | Salesforce CRM, HubSpot, Yotpo Loyalty, Smile.io |

## 3. Standard schemas

There is *no* universal retail schema, but the de facto field set per order/event is well known.

### Order (typical)
```
order_id, customer_id, created_at, currency, status,
subtotal, discount, tax, shipping, total,
billing_address, shipping_address,
payment_method, channel, source
```

### Order line item
```
order_id, line_id, sku, variant_id, product_id,
qty, unit_price, discount, tax,
fulfillment_status, return_status
```

### Catalog
```
product_id, variant_id, sku,
title, description, vendor, brand,
category_path, attributes (JSON),
price, msrp, cost,
inventory_qty, available_for_sale
```

### Event (clickstream)
```
event_id, anonymous_id, user_id, timestamp,
event_name (page_view, product_view, add_to_cart, checkout_start,
            purchase, search, ...),
properties (JSON), context (device, page, campaign, ip, ...)
```

### Schemas worth knowing

- **Schema.org Product / Offer / Review** — for SEO and structured data; useful guide for warehouse modeling.
- **GS1 GTIN** — global product number; essential for marketplaces and supply chain.
- **GS1 EPCIS** — supply-chain event format.
- **OpenRTB** — programmatic ad bidding (relevant if you do retail-media).

## 4. Cleaning particulars

### 4.1 Identity stitching

A single customer is often:

- An anonymous cookie ID for some sessions.
- A logged-in `customer_id` for others.
- A guest checkout with email but no account.
- A mobile-app `idfa` / `gaid` (the post-IDFA world makes this much sparser).
- A POS-loyalty-card swipe in-store.

Stitching these into a single identity (deterministic + probabilistic) is the single most consequential cleaning step. Mistakes propagate into LTV, churn, attribution.

### 4.2 SKU vs. variant vs. product

A "blue medium t-shirt" is a SKU. The blue colorway is a variant. The t-shirt model is a product. Catalogs frequently confuse these levels; pick one canonical and be consistent.

Categories *re-organize* monthly (merchandising team's job). Snapshot the catalog at the time of each event, or you'll have orders pointing at categories that no longer exist.

### 4.3 Orders are not events

Orders are stable, persistent objects with status transitions (placed → paid → fulfilled → returned). Events are immutable timestamped facts. Don't model orders as an event log without also keeping the snapshot — you'll lose the latest state and double-count.

### 4.4 Returns and refunds

A "purchase" event followed 30 days later by a return changes revenue, profit, LTV. Define the return window for each metric:

- Gross revenue = orders placed (always wrong as a "performance" metric).
- Net revenue = orders placed − returns within window.
- "30-day net revenue" is a common reporting standard.

### 4.5 Currency, tax, and shipping

- Multi-currency stores: convert at order time, store both `total_local` and `total_usd` (or your reporting currency).
- Tax handling differs by region (US sales tax, EU VAT, UK VAT, GST). Decide whether reported revenue is gross or net of tax.
- Shipping revenue can be reported separately or rolled in.

### 4.6 Bots and noise in clickstream

A non-trivial fraction of web traffic is bots, scrapers, and synthetic monitoring. Filter:

- Known UA strings, declared bots.
- Behavioral signatures (impossible click rates, zero scroll, headless browsers).
- Honeypot fields.

If you don't, your conversion rates and bounce rates are systematically off.

### 4.7 Attribution windows and channels

Was that sale from the email, the retargeting ad, or organic? Decision depends on:

- Look-back window (7 / 30 / 90 days).
- Touch credit (last-click, first-click, linear, U-shaped, time-decay, data-driven, MMM).
- View-through vs. click-through counting.

Document the choice per report; different stakeholders use different windows.

### 4.8 Address standardization

Shipping addresses arrive free-form. Normalize via a service (USPS, SmartyStreets, Loqate, HERE) before doing geographic analyses or matching customers across orders.

## 5. Standard analyses

### 5.1 Customer

| Analysis | Methods |
|---|---|
| RFM segmentation | Recency / Frequency / Monetary deciles |
| Customer lifetime value (CLV / LTV) | BG-NBD + Gamma-Gamma; ML survival / regression |
| Churn / repeat purchase prediction | GBM, logistic, survival |
| Next-best-action / next product | Sequence models, recsys models |
| Cohort retention | Cohort × period matrix, heatmap |

### 5.2 Product

| Analysis | Methods |
|---|---|
| Recommendation systems | Collaborative (ALS, item-CF), content (embeddings), session-based, two-tower |
| Search ranking | LTR (LambdaMART, XGBRanker, neural rerankers) |
| Demand forecasting | Hierarchical forecasting (top-down, bottom-up, MinT); Prophet, GBM on lags, TFT, N-BEATS |
| Price elasticity | Log-log regression, GBM with confounding controls, causal forests |
| Assortment / market basket | Apriori, FP-growth, embedding similarity |
| Catalog QA | Image / title duplicate detection, attribute extraction |

### 5.3 Marketing

| Analysis | Methods |
|---|---|
| Attribution | Rule-based, Markov, Shapley, MMM |
| Incrementality / lift | Geo / ghost ads, holdouts, synthetic control |
| Uplift modeling | T/S/X-learners, causal forests, CATE |
| A/B / multivariate testing | Sequential, CUPED, switchback |

### 5.4 Operations

| Analysis | Methods |
|---|---|
| Inventory optimization | Newsvendor, (s,S) policies, simulation |
| Fulfillment / warehouse routing | OR-tools, optimization |
| Fraud (chargebacks, account takeover, returns abuse) | Hybrid rule + ML, graph signals |
| Pricing optimization | Bayesian optimization, bandits, dynamic pricing |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Funnel from view → cart → checkout → purchase | **Funnel chart** with conversion rates |
| Cohort retention | **Heatmap** (rows = cohort, columns = period age) |
| Sales by category | **Treemap** or sorted bar |
| Geographic sales | **Choropleth** (equal-area) by state/region; **hex / H3** for cities |
| Top products | Sorted **bar** or **dot plot** |
| Customer LTV distribution | **Histogram on log x-axis**; **ECDF** |
| RFM segments | 3×3 grid or scatter (R vs. F, color by M) |
| Recommendation explainability | Side-by-side product cards |
| A/B test results | Posterior distribution; cumulative metric over time |
| Inventory health | Days-of-cover gauge per SKU |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **GDPR + ePrivacy (EU/UK)** | Personal data, cookie consent, profiling, DSAR, right-to-erasure |
| **CCPA / CPRA (California)** | Personal information, opt-out of sale |
| **State privacy laws (US)** | Virginia, Colorado, Connecticut, Utah, Texas, Florida, more |
| **COPPA (US)** | Under-13 data; almost any consumer site needs age gating |
| **PCI DSS** | Card storage / transmission |
| **PSD2 (EU)** | Strong customer authentication, open banking |
| **Apple ATT, Google Privacy Sandbox** | Cross-app and cross-site tracking |
| **DSA / DMA (EU)** | Platform obligations, gatekeeper rules |
| **Pricing law** | Drip-pricing rules, reference-price rules (varies by jurisdiction) |
| **Country-specific consumer protection** | Return rights (EU 14-day distance selling), warranty, labeling |

### Personalization fairness

- Price discrimination based on protected attributes (location proxy for race, device proxy for income) carries legal and reputational risk in some jurisdictions.
- Recommendations and rankings can amplify bias (popular brands crowd out marginal ones); audit by category and demographic when possible.

## 8. Public datasets

| Dataset | What |
|---|---|
| **Online Retail (UCI)** | UK gift-store transactional data |
| **Instacart Market Basket** | Order baskets, products |
| **Amazon Reviews** | Product reviews + metadata |
| **MovieLens** | Recsys benchmark |
| **Rossmann Store Sales (Kaggle)** | Daily store sales w/ promo |
| **Walmart Recruiting (Kaggle)** | Weekly sales w/ markdowns |
| **H&M Personalized Fashion Recommendations** | Real e-commerce data |
| **Otto Multi-Objective Recommender System** | Session events |
| **Criteo Display Advertising / Conversion logs** | Ad-tech adjacent |
| **Yelp / TripAdvisor reviews** | Text + ratings |
| **OpenStreetMap, Census ACS** | For geo enrichment |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars`, `duckdb` | Tabular |
| `dbt` | Warehouse modeling |
| `lifetimes`, `pymc-marketing` | CLV, MMM |
| `implicit`, `surprise`, `lightfm`, `torch-rechub`, `recbole`, `merlin` | Recsys |
| `prophet`, `statsforecast`, `darts`, `neuralforecast` | Forecasting |
| `econml`, `dowhy`, `causalml` | Uplift / causal |
| `streamlit`, `plotly Dash` | Internal dashboards |
| Snowflake / BigQuery / Redshift | Warehouse |
| `Segment` / `mParticle` SDKs | Event capture |

## 10. References

- Fader, P. & Hardie, B. *Probability Models for Customer-Base Analysis.* JIM (2009). (BG/NBD & Pareto/NBD)
- Aggarwal, C. *Recommender Systems: The Textbook*.
- Kohavi, R., Tang, D., Xu, Y. *Trustworthy Online Controlled Experiments*.
- Hyndman, R. & Athanasopoulos, G. *Forecasting: Principles and Practice* 3e (hierarchical reconciliation).
- Athey, S. & Imbens, G. — papers on causal forests and uplift.
- Schmittlein, D., Morrison, D., Colombo, R. *Counting Your Customers* (1987).
- Google Analytics 4 documentation, GA4 BigQuery export schema.
- Shopify and Amazon SP-API documentation.
