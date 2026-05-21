# Manufacturing

> **TL;DR** — Manufacturing data is mostly **high-rate sensor time series** joined to **process / quality / business records** that live in completely different systems (OT vs. IT). The hard parts are tag-name chaos across lines and vendors, multi-rate alignment (kHz sensors next to once-per-shift quality measurements), calibration drift, and the OT/IT boundary that requires its own networking, security, and protocol expertise.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **High-rate time series** | Vibration, current, voltage, pressure, temperature, flow, rotational speed (10 Hz – 50 kHz) |
| **Low-rate time series** | Setpoints, shift counters, status flags (1 / min – 1 / hour) |
| **Tabular discrete events** | Quality measurements, defect codes, downtime events, alarms |
| **Categorical** | Machine ID, line, station, recipe, product code, operator, shift |
| **Hierarchical (ISA-95)** | Enterprise → Site → Area → Work Center → Work Unit → Equipment |
| **Image** | Vision-system inspection, microscopy, X-ray / CT, thermal imaging |
| **Audio / vibration spectrograms** | Acoustic monitoring of bearings, gearboxes |
| **Tabular master data** | BOM, routings, work orders, recipes, calibration records |
| **Geospatial / graph (intra-plant)** | Plant layout, material flow paths, supplier networks |

## 2. Common sources

| System | What |
|---|---|
| **PLC / DCS** | Real-time control logic; tag values |
| **SCADA** | Supervisory layer, alarms, operator screens |
| **Historian** | OSIsoft PI (now AVEVA PI), Aveva Wonderware Historian, GE Proficy, Inductive Automation Ignition, InfluxDB |
| **MES** | Manufacturing Execution System: work-in-process, genealogy, traceability (Siemens Opcenter, Rockwell FactoryTalk, GE Plant Apps, Apriso) |
| **ERP** | SAP, Oracle, Microsoft Dynamics — work orders, BOMs, COGS, sales |
| **LIMS** | Lab Information Mgmt — quality / chemistry |
| **CMMS / EAM** | Maintenance — work orders, asset records (IBM Maximo, SAP PM, Fiix) |
| **Vision systems** | Cognex, Keyence, Basler with onboard inspection |
| **Edge / IIoT platforms** | AWS IoT SiteWise, Azure Digital Twins, GE Predix, Siemens MindSphere, PTC ThingWorx |
| **Time-series DBs** | InfluxDB, TimescaleDB, QuestDB, ClickHouse |

## 3. Standard schemas, protocols, and standards

| Standard | Used for |
|---|---|
| **ISA-95** | Enterprise / control integration hierarchy |
| **ISA-88** | Batch control |
| **OPC-UA** | Modern industrial communication, type-safe, secure |
| **OPC Classic (DA/HDA/AE)** | Legacy industrial communication (Windows-only) |
| **Modbus (TCP / RTU)** | Simple legacy device protocol |
| **EtherNet/IP (Allen-Bradley)** | Real-time control comms |
| **Profinet / Profibus (Siemens)** | Real-time control comms |
| **MQTT, MQTT Sparkplug B** | Lightweight pub/sub for IIoT, vendor-neutral payload |
| **CIP, EtherCAT, CC-Link** | Field-level protocols |
| **AutomationML, B2MML** | XML data exchange |
| **OPC UA Companion Specs** | Domain-specific (PackML, Robotics, Machine Tools) |

### Naming conventions: tag namespaces

Industrial tags look like `LINE03.STN02.MTR01.RPM` or `Plant1.Area2.Pump1.DischargePressure_PV`. There is no global standard — every plant has its own scheme. Building a canonical mapping is itself a data project.

## 4. Cleaning particulars

### 4.1 Tag-name normalization

Same physical measurement is named differently across lines, vendors, or revisions. Build:

- A **tag dictionary** mapping every physical sensor to a canonical name + units + measurement type.
- Asset hierarchy aligned to **ISA-95**.
- Unit declarations explicit (a tag's value of `25` is meaningless without `°C`, `psi`, `m/s`).

### 4.2 Calibration drift

Sensors drift. A 0.1°C/month drift on a temperature sensor adds up. Use:

- Calibration records to flag affected periods.
- Cross-sensor sanity checks (redundant measurements should agree within tolerance).
- Statistical detection (slow trend in the residual to a model).

### 4.3 Compression and dead-banding

Historians don't store every sample — they apply *swinging-door* or *boxcar* compression. The data you see is sparse and irregular. Account for:

- Apparent "flat" stretches that are really "value didn't change enough to record."
- Resampling at high rates may not give back actual high-rate data.

### 4.4 Time alignment

Sensors sample at different rates and clocks drift between PLCs. Use:

- NTP for plant-floor clock sync (PTP for sub-ms needs).
- Resampling to a common grid for analysis.
- As-of joins to attach the most recent slow-rate value (recipe, batch ID) to each fast-rate sample.

### 4.5 Gaps and downtime

A sensor gap can mean:

- The machine was off (correct value: nothing happened).
- The historian lost connection (correct value: missing / impute).
- The sensor failed (correct value: NA with diagnostic flag).

Treat them differently. Cross-reference with downtime events.

### 4.6 Genealogy and traceability

A finished product was made from intermediate batches by specific machines on specific shifts. MES stores the genealogy; joining sensor data to genealogy is what enables root-cause analysis. This join is hard because:

- Time windows are inexact ("around the 14:32 changeover").
- Multiple batches overlap in flow.
- Work-in-process can re-enter a machine.

### 4.7 Quality data is sparse and biased

You usually only sample for quality, you don't measure 100%. Defects are rare. Your training data has:

- Class imbalance (often 100:1 or worse).
- Sampling bias (operators inspect when they suspect a problem).
- Latency (lab results come hours/days after the manufacturing event).

### 4.8 Recipe / setpoint changes confound everything

A model trained on "Recipe A" doesn't transfer to "Recipe B" without explicit features for recipe and setpoint targets.

## 5. Standard analyses

| Analysis | Methods |
|---|---|
| **Predictive maintenance** | Survival, RUL regression (CNN/LSTM/Transformer), anomaly detection on vibration spectrograms |
| **Condition monitoring** | Spectral features (FFT, envelope, kurtosis), Hilbert transform, wavelet |
| **Anomaly detection** | Autoencoders, isolation forest, matrix profile, multivariate SPC |
| **Statistical Process Control (SPC)** | X̄ / R, EWMA, CUSUM, Hotelling's T² for multivariate |
| **Process capability** | Cp, Cpk, Pp, Ppk |
| **Yield optimization** | Causal forest, ANOVA on process parameters, response-surface |
| **Design of Experiments (DOE)** | Factorial, fractional factorial, Plackett-Burman, central composite |
| **Root-cause analysis** | Decision trees, Pareto on defect codes, layered regression |
| **Quality control / inspection** | CNNs for surface defect detection, anomaly detection in images |
| **OEE analysis** | Availability × Performance × Quality decomposition |
| **Energy optimization** | Regression with weather/load confounders |
| **Digital twin** | Physics-based + data-driven hybrid simulation |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Is the process in control? | **X̄ / R or I-MR control chart** with control limits |
| Drift? | **EWMA / CUSUM** |
| Multivariate process state? | **Hotelling's T² chart** |
| Top causes of scrap? | **Pareto chart** (sorted bar + cumulative line) |
| Cause-and-effect | **Ishikawa (fishbone)** diagram |
| Process flow / yield through stations | **Sankey** (flow + drop-off) |
| OEE breakdown | **Stacked bar** by reason (availability / performance / quality) |
| Vibration signature | **Spectrogram** |
| Defect map on product | Image with **annotated overlay** |
| Machine × shift performance | **Heatmap** |
| Equipment health over time | **Line + threshold band** |

## 7. Regulation and ethics

| Regulation / standard | Scope |
|---|---|
| **ISO 9001** | Quality management |
| **IATF 16949** | Automotive QMS |
| **AS9100** | Aerospace QMS |
| **ISO 13485** | Medical-device QMS |
| **ISO 14001** | Environmental management |
| **21 CFR Part 11** | FDA: electronic records / signatures (pharma, med-device, food) |
| **EU Annex 11** | EU equivalent of Part 11 for GMP |
| **GxP (GMP / GLP / GCP)** | Good Manufacturing / Lab / Clinical Practices |
| **IEC 62443** | Industrial cybersecurity |
| **NIST 800-82** | ICS security guidance |
| **REACH / RoHS** | EU substance restrictions |
| **OSHA / EU-OSHA** | Worker safety |
| **CE marking, UL, FCC** | Product certifications |

### Data governance

- **OT/IT separation** — OT networks are kept isolated; getting data out requires data diodes, brokers, or one-way replication. Ad-hoc Python on the factory floor is not appropriate.
- **Validation** — under GMP, any algorithm that influences product release must be **validated** (IQ/OQ/PQ), with documented change control.
- **Worker privacy** — GDPR applies to operator data; biometric / location tracking in plants has constraints.

## 8. Public datasets

| Dataset | What |
|---|---|
| **CMAPSS / NASA Turbofan** | Run-to-failure engine simulation |
| **NASA Bearings (IMS)** | Vibration to failure |
| **PHM Society challenges** | Various RUL / fault problems |
| **Bosch Production Line Performance (Kaggle)** | Anonymized real production data |
| **SECOM (UCI)** | Semiconductor manufacturing process / fault |
| **MIMII** | Industrial machine sound anomaly |
| **MVTec AD** | Anomaly detection on product images |
| **DAGM 2007** | Industrial texture defect |
| **C-MAPSS, FEMTO-ST bearings** | Prognostics benchmarks |
| **Tennessee Eastman Process** | Simulation benchmark for process monitoring |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars`, `pyarrow` | Tabular |
| `numpy`, `scipy.signal` | DSP, filtering, FFT |
| `tsfresh`, `tsfel`, `catch22` | Automated time-series feature extraction |
| `stumpy` | Matrix profile (anomaly / motif) |
| `pyod` | Outlier / anomaly detection |
| `scikit-survival`, `lifelines`, `pysurvival` | Survival / RUL |
| `gtda` (giotto-tda) | Topological data analysis |
| `pyspc`, `spcchart` | SPC charts |
| `dowhy`, `econml`, `causalml` | Root-cause / causal |
| `pyads` (Beckhoff), `python-snap7` (Siemens S7), `pymodbus`, `opcua-asyncio` | Industrial connectivity |
| Foundation: AVEVA PI Web API, Ignition Python scripting | Vendor-specific |

## 10. References

- Montgomery, D. C. *Introduction to Statistical Quality Control*.
- Box, G., Hunter, J. S., Hunter, W. G. *Statistics for Experimenters*.
- Randall, R. B. *Vibration-based Condition Monitoring*.
- ISA-95 / ISA-88 standards (paywall).
- Goble, W. M. *Control Systems Safety Evaluation and Reliability*.
- Saxena, A. et al. *Damage propagation modeling for aircraft engine run-to-failure simulation.* PHM (2008). (CMAPSS)
- AVEVA / OSIsoft PI documentation; OPC Foundation OPC-UA spec.
