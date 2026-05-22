# Energy and Utilities

> **TL;RT** — Energy data is dominated by high-frequency time series from smart meters (AMI), grid sensors (PMUs, SCADA), weather stations, and generation assets, all tightly coupled to market prices and weather. The distinguishing challenges are the physical constraints of the grid (power flow obeys Kirchhoff's laws, not arbitrary patterns), the tight coupling between weather and demand, and the transition from centralized fossil generation to distributed renewables (solar, wind) which introduces volatility and forecasting complexity. Short-term load forecasting (STLF) is the canonical problem, but asset failure prediction, demand-response optimization, and grid anomaly detection are increasingly important as the grid becomes more digital.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Smart meter consumption (AMI/AMR)** | 15-min, 5-min, or 1-min interval readings per customer (kWh) |
| **PMU (Phasor Measurement Unit) data** | 30–60 Hz voltage, current, frequency, angle across the grid |
| **SCADA telemetry** | Substation breaker status, transformer loading, line flows |
| **Weather** | Temperature, humidity, wind speed/direction, solar irradiance, cloud cover |
| **Generation data** | Output by source (solar kW, MW wind, hydro reservoir level, fossil plant output) |
| **Market prices** | Day-ahead, real-time LMP (Locational Marginal Pricing) per hour or 5-min |
| **Outage records** | SAIDI, SAIFI, restoration times, cause codes |
| **Asset health** | Transformer oil temperature, dissolved gas analysis (DGA), vibration |
| **Tariff / rate** | Time-of-use (TOU), critical peak pricing (CPP), demand charges |
| **Geospatial** | Transformer locations, line routes, customer addresses, service territories |

## 2. Common sources

| System | What |
|---|---|
| **AMI head-end system** | Smart meter data aggregation (Landis+Gyr, Itron, Sensus) |
| **SCADA / EMS** | Real-time grid monitoring (GE, Siemens, ABB) |
| **MDMS (Meter Data Management System)** | Meter data validation, estimation, compensation |
| **DMS (Distribution Management System)** | Distribution grid state estimation, fault location |
| **ISO/RTO portals** | Day-ahead prices, real-time prices, load, generation mix |
| **Weather services** | NOAA, NWS, ECMWF, Copernicus, commercial (DTN, The Weather Company) |
| **GIS** | Asset registry, network topology (ESRI, Oracle Utilities Network Manager) |
| **Outage management** | SAIDI/SAIFI tracking, crew dispatch |
| **DERMS (Distributed Energy Resource Management)** | Solar, battery, EV charger coordination |

### ISO/RTO data portals (US)

| ISO/RTO | Portal |
|---|---|
| **CAISO** (California) | https://www.caiso.com/market/Pages/Outlook/Default.aspx |
| **ERCOT** (Texas) | https://www.ercot.com/market-views |
| **NYISO** (New York) | https://www.nyiso.com/data |
| **PJM** (Mid-Atlantic) | https://www.pjm.com/markets-and-operations.aspx |
| **MISO** (Midwest) | https://www.misoenergy.org/markets-operations/data/ |
| **ISO-NE** (New England) | https://www.iso-ne.com/market-data-and-forecasts |

### International

| Platform | Scope |
|---|---|
| **ENTSO-E Transparency Platform** | EU/UK wholesale electricity data |
| **Nord Pool** | Nordic power market |
| **EPEX Spot** | Continental European power exchange |

## 3. Standard schemas and formats

### Smart meter reading (typical)
```
meter_id, timestamp (UTC),
interval_duration_min, reading_kwh,
reading_type (AMR, AMI, estimated),
quality_flag (GOOD, BAD, ESTIMATED, MISSING),
tariff_segment (TOU_peak, TOU_off_peak, TOU_mid_peak),
delta_kwh  # consumption in this interval
```

### LMP (Locational Marginal Price)
```
timestamp, node_id (or zone_id),
lmp ($/MWh), energy_component, congestion_component, loss_component
```

### PMU (synchrophasor — IEEE C37.118)
```
timestamp (GPS-locked), phasor_id,
voltage_magnitude, voltage_angle_deg,
frequency_hz, d_frequency_dtf,
frequencies: f_a, f_b, f_c,
quality_flags, source_id
```

### Generation by source (typical)
```
timestamp, plant_id, fuel_type (solar, wind, hydro, nuclear, coal, gas, oil),
capacity_mw, actual_output_mw, availability_factor, outage_reason
```

## 4. Cleaning particulars

### 4.1 Smart meter data quality

| Issue | Pattern | Fix |
|---|---|---|
| **Meter rollover** | Sudden drop to 0 (counter reset) | Detect jump, add counter max |
| **Negative readings** | delta_kwh < 0 | Flag as BAD, estimate |
| **Duplicate intervals** | Same timestamp twice | Deduplicate by meter_id + timestamp |
| **Missing intervals** | Gaps in time series | Interpolate or estimate (MDMS does this) |
| **Stuck meters** | Same reading for hours/days | Detect zero variance, flag |
| **Timezone / DST** | 23-hour or 25-hour days | Normalize to UTC, align to intervals |
| **Estimated vs. actual** | quality_flag = ESTIMATED | Distinguish in analysis |

### 4.2 Weather alignment

- **Nearest station:** Assign weather data from the closest station. Simple but ignores microclimates.
- **Interpolation:** Inverse-distance weighting from multiple stations.
- **NWP (Numerical Weather Prediction):** Use forecast data (ECMWF, GFS) for forecasting.
- **Solar irradiance:** Requires specific instruments (pyranometers) or satellite estimates (NSRDB).

### 4.3 Grid topology and network constraints

- **Line ratings:** Thermal limits on transmission lines. Exceeding causes tripping.
- **Transformer loading:** Overloading reduces lifespan. Monitor hotspot temperature.
- **Voltage limits:** IEEE 1547: 0.95–1.05 p.u. voltage acceptable range.
- **N-1 contingency:** Grid must survive loss of any single component.

### 4.4 Market price cleaning

- **Negative prices:** Common in day-ahead markets with high renewable penetration. Don't cap — they're real.
- **Price spikes:** Can reach $9,999/MWh (ERCOT price cap). Distinguish real spikes from data errors.
- **Missing data:** Market portals may have delayed data (real-time updates for 24–48 hours).

### 4.5 Generation forecasting

- **Solar:** Depends on cloud cover, panel angle, soiling, temperature (efficiency drops at high temp).
- **Wind:** Depends on hub-height wind speed, turbine power curve, wake effects between turbines.
- **Hydro:** Depends on snowpack, precipitation, reservoir levels, environmental flow requirements.
- **Demand:** Depends on temperature (heating/cooling degree days), day of week, holidays, events.

## 5. Standard analyses

### 5.1 Forecasting

| Analysis | Horizon | Methods |
|---|---|---|
| **STLF (Short-term load forecasting)** | 1 hour – 7 days | Regression with weather features, GBM, LSTM, TFT, N-Nets |
| **Solar forecasting** | 0–72 hours | NWP + satellite imagery, GBM, CNN-LSTM |
| **Wind forecasting** | 0–72 hours | NWP + turbine data, GBM, quantile regression |
| **Long-term demand forecast** | 1–30 years | Demographic trends, electrification, efficiency |
| **Price forecasting** | Day-ahead, real-time | GBM, regression, regime-switching models |

**STLF feature engineering:**

- Temperature (actual + forecast), humidity, wind speed, cloud cover.
- Hour-of-day, day-of-week, holiday flags.
- Lag features (load at same hour yesterday, same hour last week).
- Rolling statistics (7-day average load).
- Cooling/heating degree days.

### 5.2 Anomaly detection

| Analysis | Methods |
|---|---|
| Transformer fault detection | Dissolved gas analysis (DGA) interpretation (Duval triangle, Rogers ratio) |
| Line fault detection | SCADA event correlation, impedance-based fault location |
| Meter fraud / theft | Anomaly detection on consumption patterns (Isolation Forest, autoencoders) |
| Cyber attack detection | Unusual SCADA commands, PMU data anomalies |
| Outage prediction | Survival analysis on asset age, condition, weather exposure |

### 5.3 Demand response and optimization

| Analysis | Methods |
|---|---|
| **Demand response targeting** | Uplift modeling, propensity scoring |
| **Dynamic pricing optimization** | Constrained optimization, RL |
| **EV charging optimization** | Mixed-integer programming, RL |
| **Battery storage dispatch** | Stochastic optimization, RL |
| **DER coordination** | Distributed optimization, market mechanisms |

### 5.4 Grid analytics

| Analysis | Methods |
|---|---|
| **State estimation** | Weighted least squares, bad data detection |
| **Power flow analysis** | Newton-Raphson, DC power flow |
| **Fault location** | Impedance-based, traveling wave |
| **Voltage optimization** | OLTC (On-Load Tap Changer) control, capacitor bank switching |
| **Restoration optimization** | Graph algorithms, mixed-integer programming |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Load over time | **Line chart** with weather overlay |
| Load duration | **Load duration curve** (sorted by magnitude) |
| Hourly load pattern | **Heatmap** (day-of-week × hour-of-day) |
| Generation mix | **Stacked area chart** (solar, wind, gas, coal, nuclear) |
| Price over time | **Line chart** with price cap marker |
| Solar generation | **Line chart** (duck curve) with demand overlay |
| Transformer loading | **Line chart** with thermal limit threshold |
| Outage map | **Map** with outage count by region |
| PMU data | **Phasor diagram** (voltage vectors) |
| Weather vs. load | **Scatter** (temperature vs. kWh) with regression line |

## 7. Regulation and ethics

| Regulation / standard | Scope |
|---|---|
| **NERC CIP** (North America) | Critical infrastructure protection for bulk power system |
| **FERC** (US) | Wholesale electricity markets, open access transmission |
| **PUC / State regulators** (US) | Retail rates, service quality (SAIDI/SAIFI) |
| **EU CER (Clean Energy Package)** | Smart meter deployment, data access, customer rights |
| **GDPR** | Smart meter data is personal (reveals household activity) |
| **IEEE 1547** | Interconnection of DERs (solar, battery) |
| **IEC 61850** | Substation automation communication |
| **ISO 50001** | Energy management systems |
| **NIST 800-82** | ICS security guidance |

### Privacy

Smart meter data can reveal:

- When occupants are home or away.
- Appliance usage patterns (TV on, oven used, EV charging).
- Sleep schedules, work patterns.

Some jurisdictions (California, Texas) have specific rules on smart meter data access.

## 8. Public datasets

| Dataset | What |
|---|---|
| **EIA Open Data** (US) | Electricity data: generation, consumption, prices, grids |
| **ENTSO-E Transparency Platform** | EU wholesale: generation, demand, cross-border flows |
| **ERCOT** | Texas: load, prices, wind/solar generation, weather |
| **CAISO** | California: LMP, generation mix, solar/curtailment |
| **GEFCom** | Load, temperature, and price time series for forecasting benchmarks |
| **UCI Individual Household Electric Power Consumption** | 1-min household-level power consumption (4 years) |
| **NSRDB (NREL)** | Solar radiation data (30 years, US) |
| **NOAA/NDBC** | Marine weather (wind, pressure, wave) |
| **OpenEI** (Open Energy Information) | Aggregated energy data |
| **Kaggle: Electricity Load Diagrams** | Hourly load for 370 customers (2014–2018) |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Time-series processing |
| `prophet`, `statsforecast`, `darts`, `neuralforecast` | Load/generation forecasting |
| `scikit-learn`, `xgboost`, `lightgbm` | ML for anomaly detection, classification |
| `pytorch`, `tensorflow` | Deep learning (TFT, LSTM for forecasting) |
| `pandapower` | Power flow analysis, grid simulation |
| `PyPSA` | Power system simulation (optimization) |
| `numpy`, `scipy.signal` | Signal processing for PMU data |
| `geopandas`, `contextily` | Geospatial grid mapping |
| `h3` | Hexagonal grid for spatial aggregation |
| `optuna` | Hyperparameter optimization for forecasting |

## 10. References

- Hong, T. et al. *Probabilistic electric load forecasting: A tutorial review.* International Journal of Forecasting (2020).
- Baran, M. E. & El-Hawary, M. E. *Optimal network reconfiguration.* IEEE Proceedings (1998).
- IEEE Std C37.118 — Phasor measurement standard.
- NREL NSRDB Documentation — https://nsrdb.nrel.gov/
- EIA Open Data Documentation — https://www.eia.gov/opendata/
- ENTSO-E Transparency Platform — https://transparency.entsoe.eu/
