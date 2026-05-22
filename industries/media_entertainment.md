# Media and Entertainment

> **TL;RT** — Media and entertainment data centers on streaming sessions (play, pause, seek, stop), content metadata, and subscriber lifecycle, with heavy emphasis on recommendation systems and content valuation. The distinguishing challenges are session stitching across devices (same user on TV, phone, tablet, desktop), content-ID normalization across licensing windows and regions, and the gap between engagement metrics (watch time, completion rate) and business value (retention lift, churn reduction, licensing cost recovery). A/B testing is pervasive (player UI, recommendation algorithms, pricing), but the scale of experimentation requires careful multiple-testing correction and sequential analysis. Content recommendation is the core ML problem, but content acquisition valuation, churn prediction, and ad pricing are equally important.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Streaming session events** | Play, pause, resume, stop, seek, buffer, quality change, error |
| **Content metadata** | Title, genre, cast, crew, release date, language, rating, duration |
| **Subscriber lifecycle** | Signup, upgrade, downgrade, churn, reactivation, payment status |
| **Recommendation events** | Impression, click, play-from-recommendation, skip, thumbs-up/down |
| **Ratings and reviews** | Star ratings, text reviews, social media mentions |
| **Ad impressions** | Ad served, ad viewed, ad clicked, ad skipped, ad conversion |
| **Device / platform** | TV (Roku, Fire TV, Apple TV), mobile (iOS, Android), web, desktop |
| **Geospatial** | Subscriber location, content availability by region (licensing) |
| **Content rights** | Licensing windows, territories, exclusivity, expiry dates |
| **Quality metrics** | Bitrate, resolution, codec, buffer ratio, error rate |

## 2. Common sources

| System | What |
|---|---|
| **CDN (Content Delivery Network)** | Video delivery logs, edge caching, bandwidth |
| **Player telemetry** | Playback events, errors, quality changes (Brightcove, Bitmovin, Shaka) |
| **CMS (Content Management System)** | Content metadata, digital asset management |
| **Recommendation engine** | Impressions, clicks, rankings, model version |
| **CRM / billing** | Subscriptions, payments, plans (Stripe, Braintree, Zuora) |
| **Ad servers** | Google Ad Manager, OpenRTB exchanges, DSPs |
| **Analytics** | GA4, Adobe Analytics, Mixpanel, Amplitude, internal event platform |
| **CDP (Customer Data Platform)** | Segment, mParticle, RudderStack |
| **Social media** | Reviews, mentions, sentiment (Twitter, Reddit, IMDb) |
| **Content rights systems** | Licensing windows, territories, exclusivity |

## 3. Standard schemas and formats

### Streaming session event
```
event_id, timestamp, user_id (hashed), device_id, session_id,
event_type (play, pause, resume, stop, seek, buffer_start, buffer_end,
            quality_change, error, ad_impression, ad_click),
content_id, content_type (movie, series, episode, clip, live),
season_number, episode_number,
playback_position_sec, seek_to_position_sec,
video_bitrate_kbps, resolution, codec,
buffer_duration_sec, error_code,
platform (ios, android, web, roku, fire_tv, apple_tv),
geo_country, geo_region,
recommendation_context (rec_position, algorithm_version, impression_id)
```

### Content metadata
```
content_id, title, description,
genres (array), tags (array),
cast (array), crew (array),
release_date, content_rating (PG, R, TV-MA),
duration_sec, language,
original_language, subtitle_languages,
production_company, country_of_origin,
licensing_windows (JSON: territory, start_date, end_date, exclusivity),
cost_acquisition, revenue_share
```

### IAB standards

- **VAST (Video Ad Serving Template):** XML schema for video ad delivery.
- **VPAID (Video Player-Ad Interactive Data):** Interactive ad format (deprecated, replaced by VMAP).
- **VMAP (Video Multiple Ad Playlist):** Ad breaks within content.
- **Ads-SDK:** Server-side ad insertion (SSAI) standards.

## 4. Cleaning particulars

### 4.1 Session stitching

A single user watches on phone (commute), TV (evening), and tablet (lunch). Stitching requires:

- **Deterministic:** Logged-in user ID across devices.
- **Probabilistic:** Device fingerprinting, email matching (post-login).
- **Post-IDFA/cookie:** Limited cross-device tracking; server-side identity resolution is key.

**Session definition:**

- 30–60 minutes of inactivity → new session.
- Seek to beginning of content → new session (rewatch).
- Content change → new session (binge-watching = multiple sessions).

### 4.2 Content-ID normalization

Same content has different IDs across systems:

- Internal CMS ID.
- IMDB ID.
- TVDB ID (for series).
- Licensing partner IDs.
- Genre categorization changes (reclassified over time).

**Strategy:** Maintain a content canonical ID table with mappings to all external IDs. Snapshot the mapping at the time of each event.

### 4.3 Bot and synthetic traffic filtering

Not all streaming events are from real humans.

**Signals:**

- **Playback patterns:** 100% completion rate, no pauses, perfect linear playback (humans pause, seek, vary speed).
- **Device fingerprints:** Known bot user agents, headless browsers.
- **Volume:** Unusually high event rate per user.
- **Geographic anomalies:** Events from datacenter IPs.

**Mitigation:**

- Rule-based filtering (completion rate > 99%, no pauses).
- Anomaly detection on event rates.
- Device fingerprinting + blocklists.

### 4.4 Quality metrics cleaning

- **Bitrate fluctuations:** Normal during adaptive bitrate (ABR) streaming. Don't treat as errors.
- **Buffer ratio:** Fraction of playback time spent buffering. > 5% is poor.
- **Error codes:** Distinguish transient (network) vs. persistent (content) errors.
- **ABR ladder:** Different quality levels (480p, 720p, 1080p, 4K). Track quality transitions.

### 4.5 Recommendation context

Recommendation events are confounded by:

- **Position bias:** Items at position 1 get more clicks regardless of quality.
- **Popularity bias:** Popular content is recommended more → more plays → appears better.
- **Diversity constraints:** Recommendation engines deliberately diversify.
- **Exploration vs. exploitation:** Bandit algorithms intentionally show suboptimal recommendations.

**Analysis approach:** Always include position and algorithm version as covariates. Use position-weighted metrics (DCG, NDCG).

### 4.6 Content rights and geo-blocking

Content availability varies by territory and time:

- A show available in the US may not be available in the UK.
- Licensing windows expire — content rotates in and out of the catalog.
- Exclusive vs. non-exclusive licenses affect content strategy.

**Impact on analysis:**

- Engagement metrics are geo-specific. Don't aggregate globally without accounting for catalog differences.
- Churn analysis must control for content availability in the user's region.

## 5. Standard analyses

### 5.1 Recommendation systems

| Analysis | Methods |
|---|---|
| **Collaborative filtering** | Matrix factorization (ALS, SVD), item-item similarity |
| **Content-based** | Embeddings from metadata (genre, cast, description) |
| **Two-tower** | User tower + item tower → dot product (YouTube DNN recsys) |
| **Session-based** | GRU4Rec, SASRec, BERT4Rec (no user identity needed) |
| **Hybrid** | Collaborative + content + context (time, device, position) |
| **Ranking** | LambdaMART, XGBRanker, neural ranking (DPR, Two-Tower) |
| **Evaluation** | Recall@K, NDCG@K, MAP, coverage, diversity, novelty |

### 5.2 Content valuation

| Analysis | Methods |
|---|---|
| **Content performance** | Watch time, completion rate, new subscriber attribution |
| **Content ROI** | Acquisition cost vs. retention lift vs. revenue |
| **Content gap analysis** | Genre/region underrepresentation |
| **Original vs. licensed** | Retention impact, cost efficiency |
| **Franchise value** | Sequel/spin-off potential, merchandise |

### 5.3 Churn prediction

| Analysis | Methods |
|---|---|
| **Churn classification** | GBM, logistic regression, neural nets |
| **Churn drivers** | SHAP, feature importance, segmentation |
| **Content-driven churn** | Survival analysis with content availability as time-varying covariate |
| **Price sensitivity** | Conjoint analysis, price elasticity, coupon A/B |
| **Win-back** | Uplift modeling for re-engagement campaigns |

**Key features:**

- Watch time trends (declining?).
- Content freshness (new releases in favorite genres).
- Payment history (failed payments, plan changes).
- Support interactions.
- Competitive offers (did a competitor launch a similar show?).

### 5.4 A/B testing

| Analysis | Methods |
|---|---|
| **Experiment design** | Randomization unit (user, session, geo), sample size, duration |
| **Metric selection** | Primary (watch time, retention), guardrail (error rate, buffering) |
| **Sequential testing** | SPRT, mSPRT, Bayesian monitoring |
| **Multiple testing correction** | BH-FDR, Bonferroni, closed testing |
| **CUPED** | Variance reduction using pre-experiment baseline |
| **Novelty effects** | Hawthorne effect, peeking, carryover |

### 5.5 Ad analytics

| Analysis | Methods |
|---|---|
| **Ad viewability** | % of ad in viewport for ≥ 1 second (video) |
| **Ad completion rate** | % of ads watched to completion |
| **Ad attribution** | Last-click, MTA, MMM |
| **Ad pricing** | cCPM, cPC, cPV optimization |
| **Ad load optimization** | Balance revenue vs. user experience |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Retention over time | **Cohort retention heatmap** (signup week × week since signup) |
| Content performance | **Sorted bar** (watch time by title) |
| Engagement funnel | **Funnel chart** (impression → click → play → completion) |
| Recommendation quality | **DCG/NDCG curve** (position vs. cumulative gain) |
| Content genre mix | **Treemap** (genre × watch time) |
| Churn by cohort | **Line chart** (churn rate by signup cohort) |
| Playback quality | **Line chart** (average bitrate over time) |
| Geographic engagement | **Choropleth** (watch time by country) |
| Content catalog growth | **Stacked area** (content by type over time) |
| Ad performance | **Bar chart** (viewability, CTR, completion rate by ad type) |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **GDPR (EU)** | Subscriber data, consent for profiling, right to erasure |
| **CCPA / CPRA (California)** | Personal information, opt-out of sale |
| **COPPA (US)** | Under-13 data; kids content has strict restrictions |
| **DSA / DMA (EU)** | Platform obligations, content moderation transparency |
| **COPPA / UK Age-Appropriate Design** | Children's content design standards |
| **FCC** | Accessibility (closed captions, audio description) |
| **WCAG** | Accessibility standards for web/mobile |
| **Copyright / DMCA** | Content takedown, fair use |
| **Content rating** | Age ratings vary by country (MPAA, BBFC, FSK, CATV) |
| **Content moderation** | Hate speech, violence, misinformation rules (varies by platform) |

### Ethics

- **Recommendation bias:** Algorithms may amplify popular content, crowd out marginal creators.
- **Addiction design:** Infinite scroll, autoplay, auto-next — regulatory scrutiny increasing (EU DSA, South Korea).
- **Children's content:** Strict rules on data collection, targeted advertising, and content design.
- **Cultural representation:** Diversity in content and recommendations; audit by category and demographic.
- **Transparency:** Users should understand why content is recommended (explainability).

## 8. Public datasets

| Dataset | What |
|---|---|
| **MovieLens** | Movie ratings (1M, 25M, 27M versions) |
| **Netflix Prize** | Movie ratings (100M, anonymized) — historical |
| **IMDb datasets** | Movies, TV shows, cast, crew (IMDb CSV dumps) |
| **Spotify Million Playlist Dataset** | Playlists + track metadata |
| **Billboard Music Charts** | Weekly charts (1958–present) |
| **Kaggle streaming datasets** | Various (Netflix, YouTube, Spotify) |
| **YouTube-8M** | Video-level features + labels |
| **LSMDC (Large Scale Movie Description)** | Movie plot descriptions |
| **TVDB / TheMovieDB** | Open content metadata APIs |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Event processing |
| `implicit`, `surprise`, `lightfm`, `recbole` | Recommendation systems |
| `scikit-learn`, `xgboost`, `lightgbm` | Churn, classification |
| `lifelines`, `pymc-marketing` | Survival analysis, CLV |
| `pytorch`, `tensorflow` | Deep learning (two-tower, sequence models) |
| `evidently`, `whylogs` | Model monitoring (recommendation drift) |
| `streamlit`, `plotly Dash` | Internal dashboards |
| `dbt` | Data warehouse modeling |
| `Great Expectations` | Data quality validation |
| `statsmodels` | A/B test analysis |

## 10. References

- Adomavicius, G. & Tuzhilin, A. *Toward the next generation of recommender systems.* TKDE (2005).
- He, X. & Liao, L. *Neural Collaborative Filtering.* WWW (2017).
- YouTube DNN Recommender System — Huang, C.-Y. et al. RecSys (2019).
- Kohavi, R., Tang, D., Xu, Y. *Trustworthy Online Controlled Experiments.*
- IAB VAST / VPAID / VMAP specifications — https://www.iab.com/guidelines/video/
- Netflix Tech Blog — https://netflixtechblog.com/
