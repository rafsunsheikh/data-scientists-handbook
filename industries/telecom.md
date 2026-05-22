# Telecommunications

> **TL;DR** — Telecom data is a mix of network telemetry (cell tower KPIs, signaling records, throughput traces), subscriber behavior (CDRs, data sessions, app usage), and commercial data (billing, churn, NPS, support tickets). The distinguishing challenges are volume (billions of CDRs per day at scale), strict anonymization requirements (GDPR, data-retention laws), and the gap between network-layer metrics (signal strength, handover success) and customer-perceived quality (buffering ratio, page load time). Churn prediction and fraud detection are the two highest-impact analyses, but they require joining network, billing, and behavioral data across multiple systems that often have different schemas and update frequencies.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **CDRs (Call Detail Records)** | Per-call/event records: caller, callee, start time, duration, cell ID, type (voice/SMS/data) |
| **Data session records** | PDP context activations: APN, IP, start/end, bytes up/down, cell ID |
| **Signaling records (SS7/DIAMETER)** | Location updates, handovers, authentication events |
| **Network KPIs** | Per-cell or per-NE (network element): throughput, drop rate, handover success, congestion |
| **RF / drive-test data** | Signal strength (RSRP, RSRQ, SINR), PCI, timing advance from drive tests or crowdsourced probes |
| **Billing / rating** | Plans, tariffs, usage-based charges, prepaid balance, postpaid invoice |
| **CRM / subscriber** | KYC data, plan type, tenure, device, contract end date |
| **Support / OSS** | Trouble tickets, chat transcripts, NPS/CSAT surveys, app crash logs |
| **Geospatial** | Cell tower locations (lat/lon), coverage polygons, drive-test GPS traces |
| **Time series** | Per-cell traffic load, throughput, error rates, spectrum utilization |

## 2. Common sources

| System | What |
|---|---|
| **OSS (Operations Support Systems)** | Network performance monitoring, fault management (Netcool, Nagios, SolarWinds) |
| **BSS (Business Support Systems)** | Billing, rating, plan management (Amdocs, Netcracker, Oracle Utilities) |
| **CDR platforms** | Call/event records from MSC, SGSN, MME, PGW |
| **Probes / DPI (Deep Packet Inspection)** | Application-level traffic classification |
| **CRM** | Customer profiles, contracts, interactions (Salesforce, Siebel) |
| **NPS / survey platforms** | Customer satisfaction, churn intent |
| **App analytics** | MyCarrier, carrier apps — usage, crash reports |
| **Network element management** | RAN (NodeB, eNodeB, gNB), core (MME, SGW, PGW, UPF) |
| **Location platforms** | Cell-ID triangulation, LCS (Location Services) |
| **Fraud management** | Real-time CDR analysis, anomaly flags, SIM-box detection |

## 3. Standard schemas and formats

### CDR (typical)
```
record_id, timestamp,
msisdn (phone number), imsi, msisdn_hash,
calling_number, called_number,
call_type (MO, MT, SMS, MMS, data),
duration, start_time, end_time,
cell_id_source, cell_id_target,
apn, pdp_ip,
bytes_up, bytes_down,
charging_id, rating_group,
network_element_id, trace_id
```

### Cell KPI (typical)
```
cell_id, timestamp (15-min or 1-hour),
mcc, mnc, lac, ci,
total_attempts, success_rate,
drop_rate, handover_success_rate,
throughput_mbps_dl, throughput_mbps_ul,
connected_users, PRB_utilization,
congestion_count, error_count
```

### IETF RADIUS / Diameter
- **RADIUS (RFC 2865):** Authentication, authorization, accounting for network access.
- **Diameter (RFC 6733):** Successor to RADIUS, used in 4G/5G (Gx, Gy, Rx interfaces).

### 3GPP standards
- **TS 28.205:** Key Performance Indicators (KPIs) for NG-RAN.
- **TS 28.551:** SON (Self-Organizing Networks) data model.
- **TS 23.003:** Numbering, addressing, and identification.

## 4. Cleaning particulars

### 4.1 Anonymization and pseudonymization

CDRs contain direct identifiers (MSISDN, IMSI). Regulations require anonymization before sharing or secondary use.

**Methods:**

- **Hashing:** One-way hash of MSISDN/IMSI with salt (e.g., SHA-256). Use consistent salt for cross-record linkage.
- **Tokenization:** Replace with opaque token via lookup table (requires secure storage).
- **Generalization:** Mask last digits of phone number.
- **Differential privacy:** Add noise to aggregated statistics.

**Regulatory requirements:**

- GDPR: MSISDN and IMSI are personal data.
- ePrivacy Directive: Confidentiality of communications.
- Data retention laws (varies by country): Some require retaining CDRs for 6–24 months for law enforcement.

### 4.2 Cell ID to geolocation

Cell IDs (CI) map to physical locations, but:

- **Cell IDs are not globally unique:** Need (MCC, MNC, LAC/TAC, CI) tuple.
- **Cell locations change:** Towers are moved, sectors reoriented.
- **Estimation vs. exact:** Many datasets only have estimated cell locations (polygon, not point).
- **Multi-technology:** 2G (LAC), 3G (LAC), 4G (TAC), 5G (NCI) — different naming.

**Joining cell to location:**

```
# OMC (Operations & Maintenance Center) provides cell registry:
cell_registry[cell_id] = {lat, lon, sector_angle, range_m}

# Or estimate from drive-test / crowdsourced probe data:
# Aggregate probe reports per cell → centroid + coverage polygon
```

### 4.3 CDR deduplication and handover handling

A single data session generates multiple CDRs (PDP context activations, modifications). Deduplicate by:

- Matching charging IDs.
- Merging records with overlapping time windows.
- Deduplicating handover events (same session, different cell).

### 4.4 Time synchronization

Network elements may have clock drift. CDR timestamps should be aligned to a common time source (NTP/PTP). Check for:

- Negative durations (end time < start time).
- Timestamps in the future or past (clock reset).
- DST transitions (use UTC internally).

### 4.5 Billing data cleaning

- **Tariff changes:** A customer's plan may change mid-month. Attribute usage to the correct tariff.
- **Prepaid vs. postpaid:** Prepaid balance updates are real-time; postpaid invoicing is batch.
- **Roaming:** Roaming CDRs arrive with delay (hours to days) from partner networks.
- **Rating errors:** Mis-routed calls may be charged at wrong rate.

### 4.6 Support ticket NLP

- **Language:** Multilingual support transcripts.
- **Intent classification:** Billing, network, device, account.
- **Sentiment analysis:** Early churn indicator.
- **Entity extraction:** Phone numbers, plan names, error codes.

## 5. Standard analyses

### 5.1 Churn prediction

| Analysis | Methods |
|---|---|
| **Churn classification** | GBM (XGBoost, LightGBM), logistic regression, neural nets |
| **Churn timing** | Survival analysis (Cox PH, random survival forests) |
| **Churn drivers** | SHAP values, partial dependence, decision trees |
| **Retention targeting** | Uplift modeling (T-learner, causal trees) |
| **Customer segmentation** | RFM on usage, GMM on behavioral features |

**Key features:**

- Tenure, plan type, contract end date.
- Usage trends (declining? sudden drop?).
- Payment history (late payments, arrears).
- Support interactions (complaint frequency, NPS score).
- Network quality (drop rate in home cell, throughput).
- Competitive offers (did a competitor target this customer?).

### 5.2 Fraud detection

| Fraud type | Description | Detection |
|---|---|---|
| **SIM-box / SIM-farm** | Legit SIMs used to bypass international termination rates | Unusual call patterns, geographic mismatch |
| **IRSF (International Revenue Share Fraud)** | Fake premium-rate calls to siphon revenue | High-volume calls to specific numbers |
| **Subscription fraud** | Fake identities to obtain devices/contracts | KYC verification, device fingerprinting |
| **Usage fraud** | Stolen SIMs, cloned IMSI | Geographic impossibility (two locations in short time) |
| **Wangiri** | One-ring scam to premium numbers | Single short call to known premium number |
| **Billing fraud** | Manipulation of CDRs by insiders | Anomaly detection on billing system logs |

**Methods:**

- Rule-based (thresholds, blocklists).
- Graph analysis (call graph communities, anomaly detection).
- Isolation Forest, One-Class SVM for unsupervised anomaly detection.
- Real-time streaming (Kafka + Flink) for instant blocking.

### 5.3 Network optimization

| Analysis | Methods |
|---|---|
| Capacity planning | Traffic forecasting, queueing theory, simulation |
| Cell planning | Coverage modeling, link budget, drive-test analysis |
| Handover optimization | HO parameter tuning, A3 offset optimization |
| Congestion management | Traffic steering, load balancing, offloading |
| RF optimization | Antenna tilt/power adjustment, PCI planning |
| Root-cause analysis | Correlation of alarms, decision trees on KPIs |

### 5.4 Customer experience

| Analysis | Methods |
|---|---|
| Net Promoter Score (NPS) analysis | Survey stats, trend analysis, segmentation |
| Voice of Customer (VoC) | NLP on support transcripts, social media |
| MOS (Mean Opinion Score) prediction | Regression from network KPIs to subjective quality |
| Buffering ratio estimation | From throughput, packet loss, CDN data |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Network coverage | **Choropleth** (cell-level throughput/drop rate) + **H3 hexbin** |
| Churn rate over time | **Line chart** by cohort; **heatmap** (cohort × month) |
| Cell congestion | **Heatmap** (hour × day) per cell |
| Call volume pattern | **Line chart** (hourly) with weekday/weekend overlay |
| Customer journey | **Sankey** (plan → upgrade/downgrade/churn) |
| Network topology | **Graph** (cell-to-cell handover edges) |
| Drive-test signal | **Line chart** (signal strength along GPS trace) |
| Fraud network | **Graph** (MSISDN → called number edges, node size = call volume) |
| NPS distribution | **Bar chart** (promoter/passive/detractor) |
| KPI dashboard | **KPI tiles** + **sparklines** for trend |

## 7. Regulation and ethics

| Regulation | Scope |
|---|---|
| **GDPR (EU)** | Subscriber data, location data, consent for marketing |
| **ePrivacy Directive (EU)** | Confidentiality of communications, consent for cookies |
| **CCPA / CPRA (California)** | Personal information, opt-out of sale |
| **Data retention directives** | Varies by country — CDR retention for law enforcement |
| **FCC (US)** | Net neutrality, privacy rules, E911 |
| **Ofcom (UK)** | Service quality reporting, consumer protection |
| **BNetzA (Germany)** | Network transparency, roaming rules |
| **TRAI (India)** | Tariff regulations, DLT (distributed ledger for SMS) |
| **PCI DSS** | Payment data in billing systems |
| **ISO 27001** | Information security management |

### Ethics

- **Location tracking:** Cell-ID data reveals personal movement patterns. Strict access controls required.
- **Profiling:** Using call patterns or app usage for pricing or credit decisions may discriminate.
- **Surveillance:** Lawful intercept capabilities must be audited and logged.
- **Digital divide:** Network investment decisions should consider underserved areas.

## 8. Public datasets

| Dataset | What |
|---|---|
| **IBM Telco Churn (Kaggle)** | Synthetic telecom customer data |
| **Orange D4D Challenge** | Real CDR data from Ivory Coast (network planning) |
| **Mobicom/Campin datasets** | Smartphone sensor + network data |
| **CAIDA datasets** | Internet traffic traces |
| **Ericsson traffic datasets** | Aggregated mobility and traffic data |
| **OpenCellID** | Crowdsourced cell tower database |
| **Cellops** | Cell tower geolocation data |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | CDR and KPI processing |
| `scikit-learn`, `xgboost`, `lightgbm` | Churn, fraud classification |
| `lifelines`, `scikit-survival` | Survival analysis for churn |
| `networkx`, `igraph` | Call graph analysis |
| `geopandas`, `contextily` | Cell coverage mapping |
| `h3` | H3 hexagonal indexing for geospatial |
| `prophet`, `statsmodels` | Traffic forecasting |
| `pyfixest` | Fixed-effects regression for policy evaluation |
| `dbt` | Data warehouse modeling for billing |
| `Great Expectations` | Data quality validation |

## 10. References

- 3GPP TS 28.205: Key Performance Indicators (KPIs) for NG-RAN.
- 3GPP TS 23.003: Numbering, addressing and identification.
- Shmueli, G. *To Explain or to Predict?* Statistical Science (2010). (Churn modeling caveats)
- Gaol, U. L. et al. *Telecommunications Churn — A Literature Review.* (2016).
- FCC Customer Voice Survey documentation.
- Ericsson Mobility Report (annual) — industry trends.
