# Insurance

> **TL;RT** — Insurance data centers on policies (exposures, premiums, endorsements), claims (FNOL, payments, reserves, recoveries), and customer records, organized by line of business (P&C, life, health, specialty). The distinguishing challenges are policy versioning (endorsements and mid-term changes), exposure-on-risk alignment (premium earned vs. written), claim development triangles (reserves change over months/years as claims develop), and the actuarial rigor required for pricing and reserving. Generalized Linear Models (GLMs) are the workhorse for pricing, but machine learning is increasingly used for fraud detection, reserving, and retention. The industry is highly regulated, with rating factors subject to anti-discrimination rules and reserving governed by statutory accounting standards.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Policy** | Policy number, effective/expiry dates, premium (written/earned), coverage limits, deductibles, endorsements |
| **Exposure** | Units of risk (vehicle-years, property exposure days, payroll for workers' comp) |
| **Premium** | Gross premium, net premium, commissions, taxes, fees, surcharges |
| **Claims — FNOL** | First Notice of Loss: date of loss, cause, description, estimated reserve |
| **Claims — payments** | Payment dates, amounts, payment type (medical, property repair, indemnity) |
| **Claims — reserves** | Case reserves (adjuster estimate), IBNR (incurred but not reported) |
| **Claims — recoveries** | Subrogation, salvage, other recovery |
| **Customer / KYC** | Demographics, address, credit score (where allowed), prior claims history |
| **Third-party data** | Credit reports, MVR (motor vehicle records), property records, weather/peril maps |
| **Geospatial** | Property location, flood zones, earthquake faults, hurricane tracks |

## 2. Common sources

| System | What |
|---|---|
| **Policy admin system** | Policy issuance, endorsements, billing (Guidewire, Duck Creek, Duck Creek, Sapiens) |
| **Claims management** | FNOL, adjuster notes, payments, reserves (Guidewire ClaimsCenter, Duck Creek) |
| **Billing / rating** | Premium calculation, commission, tax, fee processing |
| **Reinsurance** | Treaty details, ceded premium, recoverables |
| **Third-party data providers** | LexisNexis, CLUE, ISO, AAA, CoreLogic, Verisk |
| **Weather / peril** | RMS, AIR, CoreLogic CAT models; NOAA, NHC (hurricane), USGS (earthquake) |
| **State DOI filings** | Rate approvals, policy forms, experience studies |
| **SLDS (state data)** | State-level insurance statistics |

## 3. Standard schemas and formats

### Policy (typical)
```
policy_id, effective_date, expiration_date,
insured_id, property_id, vehicle_id,
line_of_business (auto, home, commercial, workers_comp, umbrella, cyber),
coverage_type (liability, comprehensive, collision, dwelling, medical_payments),
limit_per_occurrence, limit_aggregate, deductible,
premium_gross, premium_net, premium_written, premium_earned,
endorsements (JSON: [endorsement_code, endorsement_date, premium_adjustment]),
rating_factor (driving_record, credit_score, construction_type, occupancy),
agent_id, state, channel (agent, direct, online, wholesale)
```

### Claim (typical)
```
claim_id, policy_id,
date_of_loss, date_of_report (FNOL), date_of_closure,
loss_cause (collision, fire, theft, water_damage, liability, medical),
severity_category (minor, moderate, major, total_loss, fatal),
case_reserve, paid_to_date, incurred (paid + reserve),
payment_history (JSON: [payment_date, payment_amount, payment_type]),
recovery_subrogation, recovery_salvage,
adjuster_id, repair_shop_id, medical_provider_id,
closure_reason (paid, reserved, open, denied, litigated)
```

### Development triangle (typical)
```
accident_year \ development_month  0   1   2   3   4   5   6   12  18  24
2018                                                       1,250,000
2019                                              890,000
2020                                       620,000
2021                                410,000
2022                         280,000
2023                  150,000
2024           55,000
```

## 4. Cleaning particulars

### 4.1 Policy versioning and endorsements

Policies change over their term through endorsements (mid-term modifications):

- **Coverage changes:** Add/remove coverage, change limits.
- **Premium adjustments:** Rating factor changes, discounts applied.
- **Insured changes:** Add/remove drivers, change address.
- **Payment changes:** Installment plan modifications.

**Cleaning strategy:**

- Maintain a **policy timeline** with effective dates for each endorsement.
- Calculate **earned premium** proportionally to the time each endorsement was in effect.
- **Exposure-on-risk:** Weight exposures by the number of days each endorsement was active.

### 4.2 Earned vs. written premium

| Metric | Definition | Use |
|---|---|---|
| **Written premium** | Total premium at policy inception | Sales reporting, commission |
| **Earned premium** | Premium earned over time (pro-rated) | Underwriting profitability, GLM rating |
| **Unearned premium** | Written − earned (liability on balance sheet) | Financial reporting |

**Example:**

```
Policy: 12-month, $1,200 premium, effective 2024-01-01
Endorsement: 2024-06-01, premium adjustment +$100

Written premium: $1,300
Earned premium (as of 2024-06-30):
  Original: $1,200 × 6/12 = $600
  Endorsement: $100 × 1/7 = $14.29
  Total earned: $614.29
```

### 4.3 Claim development triangles

Claims develop over time — reserves are set at FNOL but change as the claim progresses.

**Development periods:**

- **Reporting lag:** Days between date of loss and FNOL.
- **Resolution lag:** Days between FNOL and closure.
- **Payment lag:** Pattern of payments over time.

**Cleaning:**

- **Triangulate claims:** Organize by accident year × development month.
- **Handle zero claims:** Some cells may have zero incurred (no claims in that cell).
- **Handle outliers:** Catastrophe events (hurricane, earthquake) create spikes — consider excluding from routine reserving.
- **Closed vs. open claims:** Open claims have ongoing reserves; closed claims have final paid amounts.

### 4.4 Exposure data cleaning

- **Exposure units:** Vehicle-years (auto), exposure days (property), payroll (workers' comp), acreage (crop).
- **Mid-term changes:** Exposure must be prorated for endorsements.
- **Cancellation:** Early cancellation → actual exposure < policy term.
- **Non-renewal:** Policy expired naturally → full term exposure.

### 4.5 Third-party data integration

| Data source | Use | Cleaning challenge |
|---|---|---|
| **Credit score** | Auto pricing (where allowed) | Credit score changes over time |
| **MVR** | Auto rating (driving violations) | Violations age off; state-specific rules |
| **CLUE** | Claims history (5 years) | Report vs. paid amounts may differ |
| **CoreLogic property** | Property value, construction type, year built | Data freshness, address matching |
| **Flood / earthquake maps** | Peril exposure | Map updates, zone changes |
| **Weather data** | CAT modeling | Station proximity, interpolation |

### 4.6 Rating factor validation

Rating factors must be:

- **Actuarially sound:** Statistically significant, monotonic (where expected).
- **Non-discriminatory:** Some factors (zip code, credit score) may have disparate impact — regulated by state DOI.
- **Transparent:** Factors and weights must be filed with the state DOI.

## 5. Standard analyses

### 5.1 Pricing / GLM rating

| Component | Distribution | Link function | Purpose |
|---|---|---|---|
| **Frequency** | Poisson, Negative Binomial | Log | Claims per exposure |
| **Severity** | Gamma, Lognormal, Tweedie | Log | Cost per claim |
| **Pure premium** | Tweedie (GLM) | Log | Frequency × severity |
| **Loss ratio** | Beta (GLM) | Logit | Percentage of premium lost |

**GLM formula:**

```
g(E[Y]) = β₀ + β₁ × age + β₂ × vehicle_class + β₃ × driving_record + ...
```

**Modern extensions:**

- **GBM / XGBoost:** Non-linear effects, interactions.
- **Deep learning:** Neural nets for frequency/severity.
- **Mixture models:** High-risk vs. low-risk subpopulations.
- **Spatial models:** Geographic risk variation.

### 5.2 Reserving

| Method | Description | Stochastic? |
|---|---|---|
| **Chain-ladder** | Development factors × latest cumulative paid | No (deterministic) |
| **Bornhuetter-Ferguson** | Expected loss ratio × actual development | No |
| **B-F Chain-ladder** | Blend of chain-ladder and B-F | No |
| **Standard deviation** | Mack model (variance estimation) | Yes |
| **Tweedie GLM reserving** | Regression-based | Yes |
| **Stochastic chain-ladder** | Parametric distribution for development | Yes |
| **Mack's distribution-free** | Variance estimation without distributional assumptions | Yes |

**Chain-ladder calculation:**

```
Development factor (month i → i+1) = Σ cumulative_paid(i+1) / Σ cumulative_paid(i)
Expected ultimate = latest_cumulative × product_of_development_factors
IBNR = Expected ultimate − incurred_to_date
```

### 5.3 Fraud detection

| Analysis | Methods |
|---|---|
| **Claim-level fraud** | Rule-based (duplicate claims, timing), ML (isolation forest, XGBoost) |
| **Provider fraud** | Network analysis (provider→provider links), outlier detection |
| **FNOL fraud** | NLP on claim description, voice analysis on FNOL call |
| **Soft fraud (exaggeration)** | Severity prediction vs. actual — large residuals |
| **Hard fraud (fabrication)** | Pattern analysis (multiple claims, same repair shop) |

### 5.4 Retention and lapse

| Analysis | Methods |
|---|---|
| **Retention prediction** | Logistic regression, GBM, survival analysis |
| **Lapse prediction** | Survival analysis (Cox PH, random survival forests) |
| **Price elasticity** | GLM with price as predictor, uplift modeling |
| **Churn drivers** | SHAP, partial dependence, segmentation |
| **Retention intervention** | A/B testing discount effectiveness |

### 5.5 Catastrophe modeling

| Analysis | Methods |
|---|---|
| **CAT loss estimation** | RMS, AIR, CoreLogic models — hazard × vulnerability × exposure |
| **Scenario analysis** | Hurricane, earthquake, tornado scenarios |
| **Reinsurance optimization** | Expected loss vs. reinsurance cost |
| **Accumulation management** | Geographic concentration of exposure |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Claim development | **Development triangle** (heatmap with cumulative incurred) |
| Loss ratio over time | **Line chart** (loss ratio by accident year) |
| Frequency × severity | **Scatter** (frequency on x, severity on y, size = count) |
| Rating factor impact | **Bar chart** (relative risk by factor level) |
| Geographic risk | **Choropleth** (loss ratio by state/county) |
| Claim cause breakdown | **Stacked bar** (cause × severity category) |
| Retention by cohort | **Line chart** (retention rate by policy vintage) |
| CAT exposure | **Map** (exposure density with hazard overlay) |
| GLM diagnostics | **Residual plot** (deviance residuals vs. fitted) |

## 7. Regulation and ethics

| Regulation / standard | Scope |
|---|---|
| **State DOI (US)** | Rate approval, policy form filing, rate adequacy |
| **Solvency II (EU)** | Capital requirements, risk management, reporting |
| **IFRS 17** | Insurance contract accounting (revenue recognition, contract margins) |
| **NAIC model laws** | Uniform reporting, data standards (ACORD) |
| **Anti-discrimination** | Rating factors cannot discriminate unlawfully (race, gender) |
| **Fair Housing Act** | Property insurance cannot discriminate by neighborhood |
| **GDPR** | Customer data, right to erasure, lawful basis |
| **PCI DSS** | Payment data in billing systems |
| **SOC 2** | Information security controls |
| **ISO 31000** | Risk management framework |

### Rating factor restrictions

| Factor | Allowed? | Notes |
|---|---|---|
| **Age** | Yes (auto) | Young drivers have higher frequency |
| **Gender** | Varies by state | Banned in CA, MA, MI, NC |
| **Credit score** | Varies by state | Banned in CA, MA, HI; allowed in most states |
| **Zip code** | Varies | Disparate impact concerns; allowed in most states |
| **Marital status** | Varies | Some states ban |
| **Education / occupation** | Varies | Often correlated with protected classes |
| **Vehicle type** | Yes | Safety rating, repair cost |
| **Driving record** | Yes | Violations, accidents |

## 8. Public datasets

| Dataset | What |
|---|---|
| **French motor third-party liability (freMTPL)** | 100k+ policies, claims (actuarial benchmark) |
| **Kaggle insurance datasets** | Life insurance, health insurance, auto insurance |
| **AllState claim severity** | Claim data with text descriptions |
| **Porto Seguro safe driver** | Auto insurance fraud prediction |
| **Swiss Health Insurance** | Swiss health insurance data (microdata) |
| **NAIC Data** | Industry statistics, financial data |
| **ISO Loss Control Reports** | Property loss data |
| **CLUE database** | Claims loss underwriting exchange (proprietary) |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Data manipulation |
| `statsmodels` | GLM (Poisson, Gamma, Tweedie), survival analysis |
| `scikit-learn`, `xgboost`, `lightgbm` | Classification, regression, fraud detection |
| `lifelines`, `scikit-survival` | Survival analysis (retention, lapse) |
| `tweedie` | Tweedie GLM (pure premium modeling) |
| `pyMC` | Bayesian GLM, stochastic reserving |
| `geopandas` | Geospatial risk mapping |
| `networkx` | Fraud network analysis |
| `prophet`, `statsmodels` | Time series (loss ratio trends) |
| `dbt` | Data warehouse modeling |
| `Great Expectations` | Data quality validation |

## 10. References

- Frees, E. W. *Regression Modeling with Actuarial and Financial Applications.* Springer (2010).
- Cameron, A. C. & Trivedi, P. K. *Regression Analysis of Count Data.* Cambridge (2010).
- Mack, T. *Distribution-free calculation of the standard error of chain-ladder reserve estimates.* ASTIN (1993).
- Klugman, S., Panjer, H., Willmot, G. *Loss Models: From Data to Decisions.* 4th ed. Wiley (2019).
- IFRS 17 Insurance Contracts — IASB standard.
- Solvency II Directive 2009/138/EC.
- ACORD standards — https://www.acord.com/
