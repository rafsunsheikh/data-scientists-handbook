# Real Estate

> **TL;RT** — Real estate data centers on property records (attributes, transactions, tax assessments), market listings (MLS), and geospatial layers (zoning, parcels, schools, transit), all tied together by address standardization and property-level deduplication. The distinguishing challenges are address parsing and geocoding at scale, tracking the same property across multiple listings (relistings, price changes), reconciling MLS data with public records (discrepancies in bed/bath/sqft), and controlling for spatial autocorrelation in valuation models. Automated Valuation Models (AVMs) are the canonical application, but rental yield analysis, market-timing indicators, and portfolio risk are equally important. The data is inherently geospatial — every analysis must account for location, neighborhood effects, and spatial dependence.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Property attributes** | Bed, bath, sqft, lot size, year built, stories, garage, fireplace, pool, HVAC |
| **MLS listings** | List price, days on market, status (active, pending, sold), listing agent, broker |
| **Transactions (deeds)** | Sale price, sale date, buyer/seller, financing (cash, conventional, FHA), property type |
| **Tax assessments** | Assessed value, land value, improvement value, tax amount, exemption |
| **Rent / occupancy** | Monthly rent, lease terms, vacancy rate, occupancy rate |
| **Photos** | Interior, exterior, aerial/drone |
| **Geospatial** | Parcel boundaries, zoning, flood zones, school boundaries, transit routes |
| **Neighborhood** | Walk score, crime, amenities, demographic data |
| **Mortgage** | Loan amount, interest rate, LTV, loan type, points |
| **Market indicators** | Median price, price per sqft, months of supply, inventory, new listings |

## 2. Common sources

| Source | What |
|---|---|
| **MLS (Multiple Listing Service)** | Listings, sold/comps, days on market (local/regional MLS feeds) |
| **County recorder** | Deeds, mortgages, liens, property transfers |
| **County assessor** | Tax assessments, property attributes, parcel maps |
| **CoreLogic** | Proprietary property data, LPR (List Price Register), CoreLogic RP Data |
| **Zillow** | Zestimate, ZTRAX (historical), rent estimates, market data |
| **Redfin** | Listing data, sold data, market reports |
| **Public data** | Census ACS, BLS, BLS, EPA, FEMA flood zones, FAA airports |
| **OpenStreetMap** | Roads, buildings, POIs, transit |
| **School data** | GreatSchools, Niche, state education departments |
| **Mortgage data** | Freddie Mac PBSA, Fannie Mae DUS, HMDA (MIR) |

## 3. Standard schemas and formats

### Property (typical)
```
property_id, mls_number, address_line, city, state, zip,
latitude, longitude,
property_type (single_family, condo, townhouse, multi_family, land, commercial),
bedrooms, bathrooms, sqft_living, sqft_lot,
year_built, year_renovated, stories, garage_spaces,
fireplace, pool, basement, central_air,
zoning_code, parcel_id, legal_description,
flood_zone (X, AE, VE), school_district_id
```

### MLS listing (typical)
```
listing_id, mls_number, property_id,
status (active, pending, sold, expired, withdrawn),
list_price, original_list_price, price_changes (JSON: [date, new_price]),
days_on_market, listing_date, sold_date, sold_price,
listing_agent_id, listing_broker_id,
description, features (JSON: [pool, fireplace, hardwood_floors, ...]),
photos (JSON: [photo_url, ...]),
virtual_tour_url
```

### Transaction (deed)
```
transaction_id, property_id,
sale_date, sale_price,
financing_type (cash, conventional, FHA, VA, USDA, jumbo),
loan_amount, interest_rate, ltv_ratio,
buyer_id, seller_id,
deed_type (warranty_deed, quitclaim_deed, trust_transfer),
recording_date, recording_fee
```

### Tax assessment
```
assessment_id, property_id,
assessment_year, land_value, improvement_value, total_assessed_value,
exemptions (homestead, veteran, disabled),
tax_amount, tax_rate,
assessment_authority (county, city, special_district)
```

## 4. Cleaning particulars

### 4.1 Address parsing and standardization

Addresses arrive in free form: "1234 Main St", "1234 Main Street", "1234 Main St.", "1234 Main Str."

**Steps:**

1. **Parsing:** Extract number, street prefix, street name, street suffix, suffix direction, apartment/suite.
2. **Standardization:** USPS standardization (USPS Address Validation API, SmartyStreets, Loqate).
3. **Geocoding:** Convert address to lat/lon (Google Maps, Mapbox, ArcGIS, Nominatim).

**Python:**

```python
import usaddress

raw = "1234 MAIN ST APT 5B"
parsed = usaddress.parse(raw)
# [('Number', '1234'), ('StreetNamePreDirection', 'MAIN'), ('StreetName', 'ST'), ('OccupancyType', 'APT'), ('OccupancyIdentifier', '5B')]
```

### 4.2 Property deduplication

Same property appears in multiple sources with different IDs:

- **MLS listing** → internal MLS property ID.
- **County assessor** → parcel ID.
- **CoreLogic** → CL property ID.
- **Zillow** → Zillow property ID.

**Deduplication strategy:**

- **Deterministic:** Exact address match after standardization.
- **Probabilistic:** Fuzzy address match + lat/lon proximity + owner name.
- **Master property table:** Maintain a canonical property record with cross-references to all source IDs.

### 4.3 MLS vs. public record reconciliation

Discrepancies between MLS and county records are common:

| Field | MLS | County | Resolution |
|---|---|---|---|
| **Bedrooms** | Agent-reported | Assessor (may differ) | Use MLS for marketing, county for tax |
| **Bathrooms** | Agent-reported | Assessor (may count half-baths differently) | Same |
| **Sqft living** | Agent-reported | Assessor (measured differently) | Use MLS for valuation, county for tax |
| **Lot size** | Agent-reported | Assessor (GIS measurement) | Use assessor for legal, MLS for marketing |
| **Year built** | Agent-reported | Assessor (permits) | Use assessor, verify with permits |
| **Property type** | Agent-reported | Assessor (zoning) | Use assessor |

**Best practice:** Use assessor data for structural attributes (sqft, year built, lot size) and MLS for listing-specific data (price, status, agent info).

### 4.4 Tracking time-on-market and price changes

Properties are relisted, relisted, and relisted. Track:

- **Cumulative days on market:** From first listing to sale.
- **Price history:** Each price change creates a new record.
- **Relisting:** Same property, new listing ID — need to link via property_id.
- **Expired listings:** Property didn't sell — may be relisted at different price.

```
property_id, listing_id, status, list_price, date_listed, date_sold,
cumulative_dom, price_change_amount, price_change_pct
```

### 4.5 Spatial autocorrelation

Real estate values are spatially autocorrelated — nearby properties have similar values. Standard regression assumptions are violated.

**Solutions:**

- **Spatial lag model:** Include neighboring values as a predictor.
- **Spatial error model:** Model spatial correlation in the error term.
- **Geographically weighted regression (GWR):** Local coefficients that vary by location.
- **Neighborhood fixed effects:** Include neighborhood or zip code as a categorical feature.

### 4.6 Renovation and improvement tracking

Property values change due to renovations, not just market movements.

**Signals:**

- Building permits (county records).
- MLS listing description changes ("newly renovated kitchen").
- Photo comparison (exterior/interior changes over time).
- Tax assessment changes (improvement value increase).

## 5. Standard analyses

### 5.1 Automated Valuation Models (AVMs)

| Analysis | Methods |
|---|---|
| ** Hedonic pricing** | Regression on property attributes + location |
| **AVM (triangular)** | Regression + KNN + expert system |
| **ML-based AVM** | GBM (XGBoost, LightGBM), random forest, neural nets |
| **Spatial AVM** | GWR, spatial lag, spatial error models |
| **Time-series AVM** | Temporal features (market trend, seasonality) |
| **Uncertainty quantification** | Quantile regression, conformal prediction |

**Key features:**

- Property attributes (bed, bath, sqft, lot, year built).
- Location (lat/lon, neighborhood, zip, school district).
- Market conditions (median price, DOM, inventory at time of sale).
- Temporal (sale month, year, market cycle).
- Proximity (distance to amenities, schools, transit, negative externalities).

### 5.2 Rental analysis

| Analysis | Methods |
|---|---|
| **Rental yield** | Annual rent / property value |
| **Rent estimation** | Regression on property attributes + location |
| **Vacancy forecasting** | Time series, regression with economic indicators |
| **Rent growth** | Hedonic price index, repeat-rentals index |
| **Affordability** | Price-to-income ratio, rent-to-income ratio |

### 5.3 Market indicators

| Analysis | Methods |
|---|---|
| **Market heat index** | DOM, list-to-sale price ratio, inventory |
| **Supply/demand** | Months of supply (inventory / recent sales) |
| **Price trends** | Repeat-sales index (Case-Shiller), hedonic index |
| **Investment analysis** | Cap rate, cash-on-cash return, IRR |
| **Lead scoring** | Propensity to buy/sell (GBM, logistic regression) |

### 5.4 Portfolio risk

| Analysis | Methods |
|---|---|
| **Concentration risk** | Geographic, property-type, tenant-type concentration |
| **CAT risk** | Flood, earthquake, hurricane exposure |
| **Stress testing** | Scenario analysis (price decline, rate increase) |
| **Value at Risk (VaR)** | Historical simulation, parametric VaR |
| **Diversification** | Correlation matrix across markets |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Price by location | **Choropleth** (median price by zip/neighborhood) |
| Days on market | **Histogram** or **box plot** by property type |
| Price trends | **Line chart** (median price over time) |
| Comparable sales | **Grid** (side-by-side property cards with photos + key metrics) |
| Rent vs. price | **Scatter** (monthly rent vs. sale price) |
| Market heat | **KPI tiles** (median price, DOM, inventory, months of supply) |
| Property attributes | **Box plot** (sqft by neighborhood) |
| Geospatial risk | **Map** (flood zone overlay + property locations) |
| Price per sqft | **Choropleth** or **dot map** |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **Fair Housing Act (US)** | Prohibits discrimination in housing based on race, color, religion, sex, national origin, familial status, disability |
| **ECOA (US)** | Equal Credit Opportunity Act — lending discrimination |
| **GDPR (EU)** | Property data (personal if linked to individual) |
| **California Prop 65** | Environmental hazard disclosure |
| **FEMA flood maps** | Flood zone disclosure requirements |
| **State disclosure laws** | Lead paint, earthquake, natural hazard disclosure |
| **Anti-discrimination in pricing** | Algorithms cannot discriminate — audit AVMs for disparate impact |
| **CCPA / CPRA (California)** | Property data privacy |
| **RESPA (US)** | Real estate settlement procedures |
| **HMDA (US)** | Mortgage lending data reporting |

### Ethics

- **Algorithmic bias:** AVMs trained on historical data may perpetuate historical discrimination (redlining effects).
- **Fair housing:** Pricing algorithms must not proxy for protected characteristics (zip code, school district).
- **Transparency:** AVM uncertainty estimates should be communicated to users.
- **Data accuracy:** Incorrect property attributes (sqft, bed/bath) can affect property taxes and sale prices.

## 8. Public datasets

| Dataset | What |
|---|---|
| **Zillow Research** | Zestimate, rent estimates, market data (API) |
| **FHFA HPI** | House Price Index (repeat-sales) |
| **US Census ACS** | Housing characteristics, rent, mortgage |
| **NYC Open Data** | Property tax, building permits, DOB jobs |
| **LA Open Data** | Property, permit, zoning data |
| **Chicago Data Portal** | Property, permit, crime, schools |
| **CoreLogic LPR** (sample) | Listing data (academic license) |
| **FRED** | Mortgage rates, housing starts, building permits |
| **Case-Shiller** | Repeat-sales price index (via FRED) |
| **OpenStreetMap** | Roads, buildings, POIs |
| **FEMA flood maps** | Flood zone data |
| **EPA Risk Screening** | Environmental hazards near properties |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Data manipulation |
| `geopandas`, `contextily` | Geospatial analysis and mapping |
| `scikit-learn`, `xgboost`, `lightgbm` | AVM, classification |
| `statsmodels` | Spatial regression (SAR, SEM) |
| `pysal` | Spatial statistics (GWR, Moran's I) |
| `usaddress` | Address parsing |
| `geopy` | Geocoding (multiple providers) |
| `h3` | Hexagonal indexing for spatial aggregation |
| `folium`, `plotly` | Interactive maps |
| `matplotlib`, `seaborn` | Publication-quality charts |
| `dbt` | Data warehouse modeling |
| `Great Expectations` | Data quality validation |

## 10. References

- Cropper, M. et al. *Hedonic Price Models: A Review.* In Handbook of Regional and Urban Economics (1997).
- Case, K. & Shiller, R. *The Efficiency of the Market for Single-Family Homes.* AER (1987). (Case-Shiller methodology)
- McMillen, D. *Spatial Econometrics for Real Estate.* Journal of Real Estate Literature (2003).
- Zillow Research Documentation — https://www.zillow.com/research/data/
- FHFA House Price Index Documentation — https://www.fhfa.gov/DataTools/Downloads/Pages/House-Price-Index-Downloads.aspx
