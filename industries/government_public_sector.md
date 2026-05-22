# Government and Public Sector

> **TL;RT** — Government data is characterized by statutory mandates (census, surveys, public records), geographic granularity (down to census tract or block), frequent schema changes (redistricting, reclassification), and strict accessibility and transparency requirements. The data spans administrative records (tax, benefits, licensing), official statistics (census, labor, health), public safety records (crime, calls for service), and open data portals. The distinguishing challenges are geography changes over time (redistricting, county boundary changes), weighting and post-stratification for survey data, code-list translations (old vs. new classification systems), and the tension between open data mandates and privacy protection. Analysis is often subject to fairness constraints, transparency requirements, and FOIA / sunshine laws.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Census / demographic** | Population counts, age, race, ethnicity, sex, household size, housing |
| **Administrative records** | Tax returns, benefit claims, licensing, permits, registrations |
| **Vital statistics** | Births, deaths, marriages, divorces |
| **Public safety** | Crime reports, calls for service, arrests, court records |
| **Educational attainment** | School enrollment, graduation rates, test scores, funding |
| **Environmental monitoring** | Air quality, water quality, radiation, noise |
| **Economic** | Employment, wages, business formation, GDP (local) |
| **Transportation** | Transit ridership, traffic counts, road conditions |
| **Geospatial** | Boundaries (census tract, block, precinct), parcel data, zoning |
| **Open data / FOIA** | Published datasets, released records |

## 2. Common sources

| Source | What |
|---|---|
| **Central statistical agencies** | US Census Bureau, Eurostat, ONS (UK), INSEE (France), ABS (Australia) |
| **Open data portals** | data.gov (US), data.gov.uk, data.europa.eu, city portals (data.nyc.gov, data.lacity.org) |
| **FOIA / public records requests** | Agency records released under freedom-of-information laws |
| **Federal agencies** | BLS, BEA, CDC, EPA, DOT, HUD, DOJ, IRS (aggregated) |
| **State / provincial** | State health departments, transportation departments, education departments |
| **Local / municipal** | City services, police departments, schools, utilities |
| **International organizations** | World Bank, OECD, UN, IMF, EU |
| **Court records** | PACER (US federal), state court databases |
| **Legislative records** | Bill text, voting records, committee hearings |

## 3. Standard schemas and formats

### Census geography codes
```
# US Census geography hierarchy:
state_fips: "06"          # California
county_fips: "085"        # San Francisco County
tract: "101000"           # Census tract
block: "1001"             # Census block
block_group: "2"          # Block group
precinct: "001"           # Voting precinct
ZCTA: "94102"             # ZIP Code Tabulation Area
```

### Crime reporting (NIBRS)
```
incident_id, date, time,
location_type (street, business, residence, school),
latitude, longitude,
offense_type (homicide, rape, robbery, aggravated_assault, burglary, larceny, arson),
victim_count, victim_age, victim_race,
offender_count, offender_race,
weapon_used, property_loss,
clearance_status (arrest, exceptional, under investigation)
```

### UCR (Uniform Crime Reporting — legacy)
```
reporting_agency, report_date,
part1_offenses (murder, rape, robbery, aggravated_assault, burglary, larceny-theft, motor_vehicle_theft),
part2_offenses (fraud, vandalism, drug_violation, disorderly_conduct, DUI),
population_served, arrest_data
```

### Birth / death records (US Standard Certificate)
```
record_id, date_of_event,
parent_names, parent_ages, parent_races,
gestational_age, birth_weight, birth_place,
attending_physician, cause_of_death (ICD-10 code),
infant_death_flag, maternal_death_flag
```

## 4. Cleaning particular

### 4.1 Geography changes over time

Census boundaries change with each decennial census and with redistricting.

**Key issues:**

- **Decennial census:** Tract and block boundaries updated every 10 years. Data from 2010 and 2020 are NOT directly comparable by geography.
- **ZCTA changes:** ZIP Code Tabulation Areas are approximations, not real boundaries. They change infrequently but inconsistently.
- **County boundary changes:** Counties merge, split, or rename (e.g., Alaska boroughs, independent cities in Virginia).
- **Redistricting:** Legislative district boundaries change every 10 years.

**Solutions:**

- Use **PUMS (Public Use Microdata Samples)** — individual records with geography codes that are stable across censuses.
- Use **crosswalks** — Census provides relationship files between old and new geographies.
- Use **historic GIS** — MHC (Municipal Historical GIS) or NHGIS (National Historical GIS) for historical boundaries.

### 4.2 Code-list translations

Classification systems change over time.

| System | Versions | Changes |
|---|---|---|
| **NAICS** (industry) | 1997, 2002, 2007, 2012, 2017 | Sector redefinitions, new industries |
| **SOC** (occupation) | 1990, 2000, 2010, 2018 | Occupation reclassification |
| **ICD-10** (disease) | ICD-10 (1990), ICD-11 (2022) | Major code restructuring |
| **CPT** (medical procedure) | Annual updates | New codes, deleted codes |
| **FIPS** (geography) | Varies | County merges, new entries |
| **ANSI Z80** (state codes) | Stable | Rare changes |

**Solutions:**

- Maintain a **code mapping table** in version control.
- Use **hierarchical codes** (first 2–3 digits often stable).
- Document the version of each code system used in each dataset.

### 4.3 Weighting and post-stratification

Government surveys use complex sampling designs. Ignoring the design invalidates standard errors.

**Key concepts:**

- **Sampling weight (wt):** Inverse of selection probability.
- **Stratum ID:** Which stratum the sample came from.
- **PSU (Primary Sampling Unit):** The cluster selected in the first stage.
- **Replicate weights:** BRR (Balanced Repeated Replication) or jackknife weights for variance estimation.

**Python (samplics):**

```python
from samplics.survey_designs import HorvitzThompson
from samplics.population_totals import PopulationTotals

design = HorvitzThompson(
    data=df,
    sample_weight="sampling_weight",
    strata="stratum_id",
    psu="psu_id",
)
estimate = design.get_total("income")
print(estimate.summary())  # includes correct standard error
```

### 4.4 Missing-by-design fields

Some fields are intentionally missing for certain records:

- **Income** not asked in short-form Census (only long-form / ACS).
- **Detailed occupation** only for employed respondents.
- **Voting history** not collected in all surveys.
- **Health details** only for respondents who screened positive.

Treat these as **structural missingness**, not random. Don't impute — the field was not asked.

### 4.5 PII and data use agreements

Government data often has strict access controls:

- **Census PUMS:** Requires registration, data security plan, approved project description.
- **IRS data:** Cannot be used for commercial purposes; strict penalties for misuse.
- **Health records:** HIPAA-protected; requires IRB or DUA.
- **Crime data:** May contain victim information (redaction required).

### 4.6 Accessibility requirements

US federal agencies must comply with:

- **Section 508** (Rehabilitation Act): Digital accessibility standards.
- **WCAG 2.1 AA:** Web content accessibility.
- **E-Government Act:** Transparency and public access.

Data visualizations must be:

- Color-blind friendly.
- Text alternatives for charts.
- Screen reader compatible.
- Available in multiple languages (where required).

## 5. Standard analyses

### 5.1 Policy evaluation

| Analysis | Methods |
|---|---|
| **Program impact** | RDD, matching (PSM, entropy balancing), DiD, IV |
| **Cost-benefit analysis** | CBA, CEA (cost-effectiveness analysis) |
| **Equity analysis** | Disparate impact, Gini coefficient, concentration index |
| **Demand forecasting** | Time series, regression with policy dummies |
| **Risk targeting** | Predictive modeling (within fairness constraints) |

### 5.2 Demographic analysis

| Analysis | Methods |
|---|---|
| **Population projections** | Cohort-component method, Markov models |
| **Migration analysis** | Net migration, origin-destination flows |
| **Housing affordability** | Price-to-income ratio, rent burden |
| **Educational outcomes** | Value-added models, cohort tracking |

### 5.3 Public safety

| Analysis | Methods |
|---|---|
| **Crime trend analysis** | Time series, seasonal decomposition |
| **Hotspot detection** | Kernel density, scan statistics (SaTScan) |
| **Predictive policing** | Risk terrain modeling (controversial — fairness concerns) |
| **Use-of-force analysis** | Rate per 1,000 encounters, demographic breakdown |
| **Recidivism** | Survival analysis, risk scoring (fairness review required) |

### 5.4 Environmental analysis

| Analysis | Methods |
|---|---|
| **Air quality trends** | Time series, spatial interpolation (kriging) |
| **Environmental justice** | Disparity analysis (pollution exposure by race/income) |
| **Climate risk** | Flood maps, heat island analysis |
| **Water quality** | Time series, threshold exceedance |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Population by geography | **Choropleth** (equal-area projection) |
| Demographic change | **Small-multiple time series** by region |
| Crime trends | **Line chart** with confidence bands |
| Budget allocation | **Treemap** or **stacked bar** by department |
| Service delivery | **Flow map** (requests → resolution) |
| Equity analysis | **Box plot** (outcome by demographic group) |
| Performance metrics | **KPI tiles** with trend sparklines |
| Public feedback | **Word cloud** (NLP on comments), **sentiment trend** |
| Accessibility | All charts must be **color-blind safe** + text alternatives |

## 7. Regulation and ethics

| Regulation / principle | Scope |
|---|---|
| **FOIA (US)** | Freedom of Information Act — public access to agency records |
| ** sunshine laws** | Open meetings, public records (varies by state/local) |
| **Section 508** | Digital accessibility (federal) |
| **WCAG 2.1** | Web accessibility standards |
| **E-Government Act** | Transparency, public access |
| **HIPAA** | Health data privacy |
| **COPRA (Children's Online Privacy)** | Data collection from minors |
| **GDPR** | EU public sector data processing |
| **Open Data Policy (US, 2013)** | Federal agencies publish machine-readable data |
| **Fair Housing Act** | Non-discrimination in housing data / algorithms |
| **Title VI** | Non-discrimination in federally funded programs |

### Ethics

- **Algorithmic fairness:** Government algorithms (risk scoring, benefit allocation) are subject to fairness audits.
- **Transparency:** Source code and data behind government algorithms may be public record.
- **Data minimization:** Collect only what is necessary.
- **Informed consent:** Surveys and studies require IRB approval.
- **Equity:** Analysis should examine impacts across demographic groups.

## 8. Public datasets

| Dataset | Description |
|---|---|
| **US Census PUMS** | Individual microdata (1%, 5% samples) |
| **American Community Survey (ACS)** | Annual US demographics |
| **BLS (Bureau of Labor Statistics)** | Employment, wages, inflation |
| **CDC WONDER** | Mortality, natality, cancer |
| **NHANES** | Health and nutrition (with exams) |
| **NIBRS crime data** | FBI crime reports (detailed) |
| **EPA Envirofacts** | Environmental compliance data |
| **EDD (Education Data Dashboard)** | School performance, funding |
| **data.gov** | 300k+ US federal datasets |
| **Eurostat** | EU statistics |
| **OECD Data** | International statistics |
| **World Bank Open Data** | Development indicators |
| **UNdata** | Global statistics |
| **PACER** | US federal court records |
| **OpenSecrets** | Campaign finance, lobbying |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Data manipulation |
| `samplics` | Survey-weighted statistics |
| `statsmodels` | Regression with survey weights |
| `geopandas`, `contextily` | Geospatial analysis |
| `pysal` | Spatial statistics (hotspot detection, kriging) |
| `matplotlib`, `seaborn` | Publication-quality charts (accessibility) |
| `plotly` | Interactive dashboards |
| `streamlit`, `dash` | Internal dashboards |
| `dbt` | Data warehouse modeling |
| `great_expectations` | Data quality validation |
| `shapely`, `fiona` | Geospatial geometry |
| `SaTScan` | Spatial scan statistics (crime, disease) |

## 10. References

- US Census Bureau Documentation — https://www.census.gov/
- BLS Handbook of Methods — https://www.bls.gov/opub/hom/
- NHGIS (National Historical GIS) — https://www.nhgis.org/
- ICD-10-CM/PCS Documentation — https://www.cdc.gov/nchs/icd/
- FOIA Guide — https://www.justice.gov/oip/foia-guide
- Section 508 Accessibility Standards — https://www.section508.gov/
- Wilkinson, M. et al. *The FAIR Guiding Principles.* Scientific Data (2016).
