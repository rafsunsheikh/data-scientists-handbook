# Marketing and Advertising

> **TL;DR** — Marketing data is a million events at the top (impressions, clicks) narrowing to a few hundred at the bottom (conversions, revenue). The whole industry runs on imperfect joins: ad impression → click → site visit → conversion, often across devices and platforms. As tracking deprecates (ATT, cookie loss, Privacy Sandbox), the field is moving from deterministic last-click attribution to probabilistic incrementality and Media Mix Modeling (MMM). Knowing what causes what, not just what correlates, is increasingly central.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Tabular events at huge scale** | Impressions, clicks, conversions (billions/day at scale) |
| **Tabular aggregated** | Spend, impressions, clicks, conversions per (campaign × ad × placement × date) |
| **Categorical (high-cardinality)** | Campaign / ad-group / creative / placement IDs; audience segments |
| **Categorical (low-cardinality)** | Channel, device, OS, country, age bucket |
| **Time series** | Daily / hourly spend and outcomes per campaign |
| **Text** | Ad copy, search queries, landing pages |
| **Image / video** | Ad creatives, dynamic feed creatives |
| **Geospatial** | Geo-targeting, geo-lift studies, location-based audiences |
| **Identifiers (sensitive)** | Cookies, mobile ad IDs, hashed emails, household IDs |
| **Graph** | User-product affinity, lookalike networks |

## 2. Common sources

| Layer | Source |
|---|---|
| **Ad platforms (walled gardens)** | Google Ads, Meta Ads (Facebook + Instagram), TikTok Ads, LinkedIn Ads, X Ads, Snapchat, Pinterest, Reddit, Amazon Ads |
| **DSPs** | The Trade Desk, DV360, Amazon DSP, MediaMath, Yahoo DSP |
| **SSPs / ad exchanges** | Google AdX, OpenX, Magnite, PubMatic, Index Exchange |
| **MMPs (mobile measurement)** | AppsFlyer, Adjust, Kochava, Branch, Singular |
| **Web analytics** | GA4 (BigQuery export), Adobe Analytics, Snowplow, Heap, Mixpanel, Amplitude |
| **Tag managers** | GTM, Tealium |
| **CDPs** | Segment, mParticle, RudderStack, Tealium, ActionIQ, Treasure Data |
| **CRMs** | Salesforce, HubSpot, Microsoft Dynamics, Marketo |
| **ESPs / messaging** | Klaviyo, Braze, Iterable, Customer.io, Mailchimp, SendGrid |
| **Email deliverability** | SparkPost, SendGrid, MessageBird stats |
| **Brand / survey** | Kantar, Nielsen, YouGov |
| **Search / SEO** | Search Console, Ahrefs, SEMrush, Moz |

### "First-party" vs. "second / third party"

- **1P** — your own customer data; the only privacy-resilient long-term asset.
- **2P** — a partner's 1P shared with you.
- **3P** — purchased / inferred from data brokers; legally and ethically increasingly fraught.

## 3. Standard schemas

There is no universal schema, but the structure of "impression → click → conversion" tables is consistent.

### Impression (typical)
```
impression_id, timestamp,
ad_id, campaign_id, ad_group_id, creative_id, placement_id,
device_id (idfa/gaid/cookie), user_agent, ip_hash,
geo (country, region, city), site, page_url, position,
bid_price, win_price, currency
```

### Click
```
click_id, impression_id (link), timestamp, ad_id, device_id
```

### Conversion
```
conversion_id, timestamp, user_id (post-login), device_id,
event_type (purchase, signup, install, lead),
value, currency, attribution_window, attributed_touchpoints (JSON)
```

### IAB / OpenRTB
- **OpenRTB 2.x / 3.x** — bid request/response schema for programmatic.
- **IAB AdCOM** — content categories.
- **IAB GVL** — vendor list for consent (TCF).

### MMP postbacks
Vendor-specific JSON describing app install / in-app events attributed to a campaign. Each MMP has its own schema; macros (`{publisher_id}`, `{campaign_id}`) get expanded server-side.

## 4. Cleaning particulars

### 4.1 Identity in the post-cookie / post-IDFA world

The deterministic identifier you used to rely on (cookies, IDFA, GAID) is increasingly unavailable:

- **Apple ATT (2021)** — explicit opt-in for IDFA; opt-in ~25% globally.
- **Safari ITP** — third-party cookies gone, first-party cookies limited.
- **Chrome Privacy Sandbox** — third-party cookies phased out (rolling); replaced by Topics, Protected Audiences, Attribution Reporting.
- **Firefox ETP** — third-party cookies blocked by default.

Practical implications:

- More users are "untrackable" → measurement gaps.
- Probabilistic attribution and aggregated measurement (SKAdNetwork, AEM, Privacy Sandbox APIs) are now first-class.
- Conversion APIs (CAPI), server-side tagging, and consent-mode are standard.

### 4.2 Attribution windows

A conversion happens on day 14. The user clicked an ad on day 1, saw an ad on day 7, and converted organically from a search on day 14. Who gets credit depends on:

- **Window length** — 1 / 7 / 28 days are typical click windows; 1 day for view.
- **Touch model** — last-click, first-click, linear, time-decay, U-shaped (W-shaped), data-driven, position-based.

Document the windows and models per metric. Different stakeholders use different.

### 4.3 Bot and fraud filtering

Ad fraud is industry-scale. Filter:

- IVT (Invalid Traffic) — IAB MRC standards (SIVT / GIVT).
- Spider/bot UAs, datacenter IPs, headless browsers.
- Behavioral signatures (impossible click-through rates, identical timing).
- MMP-flagged fraudulent installs.
- Cross-checking platform-reported vs. server-verified events.

### 4.4 Currency and reporting

- Spend reported in account currency.
- Conversion value in store currency.
- A "global ROAS" needs FX normalization to a reporting currency on a deterministic date.

### 4.5 Cross-device / cross-channel stitching

Same user on phone (Meta in-app), desktop (Google search), and tablet (email). Without a logged-in identity, stitching is probabilistic. Walled gardens (Meta, Google) do their own stitching in their reports; cross-platform stitching is the analyst's responsibility.

### 4.6 Platform reporting differences

The same campaign can show different numbers on Meta and your warehouse:

- View-through windows differ.
- "Conversion" definitions differ.
- Time-of-conversion vs. time-of-impression dating.
- Modeled conversions (now standard on Meta/Google).

Always reconcile and document the gap.

### 4.7 Survivorship / selection bias in audiences

Lookalike and retargeting audiences are based on users who already converted or interacted. Don't naively claim those audiences "caused" subsequent conversions — they were selected on engagement.

## 5. Standard analyses

### 5.1 Measurement

| Analysis | Methods |
|---|---|
| **Multi-touch attribution (MTA)** | Rule-based, Markov chain, Shapley, data-driven attribution (DDA) |
| **Media Mix Modeling (MMM)** | Bayesian regression (PyMC-Marketing, Robyn), saturation + adstock transforms |
| **Incrementality / lift testing** | Geo experiments, ghost ads, conversion lift, synthetic control, switchback |
| **A/B / multivariate testing** | Frequentist, Bayesian, sequential (SPRT, mSPRT), CUPED variance reduction |
| **Uplift modeling** | T-learner, S-learner, X-learner, R-learner, causal forests |

### 5.2 Audience and targeting

| Analysis | Methods |
|---|---|
| Customer segmentation | k-means / GMM on engagement features; latent class |
| Lookalike modeling | Two-tower embeddings, similarity ranking |
| Propensity / next-best-offer | GBM, deep models |
| Audience overlap / reach | Set theory, log-frequency tables |
| Frequency capping optimization | Marginal-utility-per-impression |

### 5.3 Creative and content

| Analysis | Methods |
|---|---|
| Ad creative classification / tagging | Vision / multimodal models on creatives |
| Copy generation / variant testing | LLMs + multi-armed bandits |
| Landing-page optimization | A/B + heatmaps |

### 5.4 Bidding and pacing

| Analysis | Methods |
|---|---|
| Bid optimization | RL, contextual bandits, value-based bidding |
| Pacing | Control theory; throttle to spend evenly across the day |
| Budget allocation | Constrained optimization across channels |

### 5.5 SEO and organic

| Analysis | Methods |
|---|---|
| Keyword ranking and gap | Crawl + position tracking |
| Topical authority | Embedding clustering of pages |
| Internal-link optimization | Graph centrality |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Funnel performance | **Funnel chart** with conversion rates |
| Spend vs. outcome | **Scatter** + size for spend |
| Channel mix | **Stacked bar** of spend / conversions by channel over time |
| MMM contribution | **Stacked area** of estimated contribution per channel |
| Attribution comparison | **Grouped bar** across attribution models |
| Geo lift | **Map** of treatment vs. control regions |
| Test results | Posterior distribution; cumulative metric over time |
| Audience overlap | **Venn-like** or **upset plot** |
| Ad creative leaderboard | Sorted table with thumbnails + metric |
| Pacing | Spend curve vs. linear pacing line |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **GDPR (EU) + ePrivacy** | Personal data, consent for tracking, profiling rights |
| **EU TCF v2** | Industry consent framework for ad tech |
| **CCPA / CPRA (California)** | Sale/share opt-out, sensitive personal information |
| **US state laws** | VA, CO, CT, UT, TX, FL, OR, MT, IA, ... (rapidly proliferating) |
| **COPPA (US)** | Under-13 cannot be tracked / profiled |
| **CAN-SPAM (US), CASL (Canada), PECR (UK)** | Email rules |
| **Apple ATT, Privacy Manifests** | App tracking controls |
| **Google Privacy Sandbox, Topics, Protected Audiences, Attribution Reporting** | Replacement for third-party cookies |
| **DSA / DMA (EU)** | Platform / gatekeeper rules; transparency on ads |
| **AI Act (EU)** (incoming) | Restrictions on emotion / biometric inference, deepfakes |

### Specific ad-content rules

- Health, finance, gambling, alcohol, political ads have **special** rules platform-by-platform and jurisdiction-by-jurisdiction.
- Targeting on protected attributes is increasingly restricted (housing, employment, credit — HUD action against Meta).
- Deceptive practices are FTC-enforced.

### Privacy-preserving measurement

- **Differential privacy** — Apple's SKAdNetwork, Privacy Sandbox APIs apply DP.
- **Aggregation thresholds** — minimum conversion counts before reporting.
- **Clean rooms** — Google ADH, Amazon Marketing Cloud, AWS / Snowflake / Databricks data clean rooms.

## 8. Public datasets

| Dataset | What |
|---|---|
| **Criteo Display Advertising (Kaggle, 2014)** | Click prediction at scale |
| **Criteo Conversion Logs / Attribution** | Counterfactual attribution |
| **Avazu CTR (Kaggle)** | Mobile ad CTR |
| **Outbrain Click Prediction (Kaggle)** | Native ads |
| **iPinYou** | Programmatic bidding |
| **YouTube-8M, MovieLens** | Recsys-adjacent |
| **Twitter Recsys Challenge** | Engagement prediction |
| **Yahoo R6A** | News article recommendation (bandit benchmark) |
| **GA4 sample dataset (BigQuery)** | Web analytics structure |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pymc-marketing` | Bayesian MMM, CLV, audience |
| `lightweight_mmm`, `Robyn` (R, callable) | MMM |
| `econml`, `dowhy`, `causalml`, `causalpy` | Causal / uplift / synthetic control |
| `pyfixest`, `linearmodels` | Fixed-effects / panel models for geo studies |
| `prophet`, `statsforecast`, `darts` | Forecasting |
| `lifetimes` | BG-NBD CLV |
| `optuna`, `hyperopt`, `bayes_opt` | Bandits / hyperparameter |
| `vowpal_wabbit` | Online learning / contextual bandits |
| `dbt` | Warehouse modeling |
| Google `bq` / Meta CAPI / TikTok Events API SDKs | Conversion uploads |

## 10. References

- Kohavi, R., Tang, D., Xu, Y. *Trustworthy Online Controlled Experiments.*
- Jin, Y. et al. *Bayesian Methods for Media Mix Modeling with Carryover and Shape Effects.* Google (2017).
- Chen, X. et al. *Lift Studies and Causal Inference at Scale.* Meta / Google whitepapers.
- Athey, S. & Wager, S. — causal-forest papers.
- IAB Tech Lab specs — OpenRTB, AdCOM, Open Measurement.
- Google Privacy Sandbox documentation.
- Apple SKAdNetwork documentation, Privacy Manifests guidance.
- Meta Aggregated Event Measurement (AEM) documentation.
