# Industries

> **Status: all filled.** All fifteen industry chapters are written in depth. Each covers data types, sources, standard schemas, cleaning particulars, standard analyses, visualizations, regulation, public datasets, and tools.

Every industry has its own data shapes, primary sources, regulatory constraints, standard analyses, and gotchas. This section is a map of *where* the abstract data-type / cleaning / visualization knowledge lands in practice.

## Index

| Industry | Status | Distinguishing data |
|---|---|---|
| [healthcare.md](healthcare.md) | filled | EHR, claims, imaging, genomics, wearables; PHI / HIPAA |
| [finance.md](finance.md) | filled | Tick / OHLCV, transactions, balances, KYC, risk models |
| [retail_ecommerce.md](retail_ecommerce.md) | filled | Orders, sessions, catalog, inventory, returns |
| [manufacturing.md](manufacturing.md) | filled | Sensor / SCADA, MES, quality, yield, supply chain |
| [marketing_advertising.md](marketing_advertising.md) | filled | Impressions, clicks, attribution, audience, A/B |
| [telecom.md](telecom.md) | filled | CDRs, churn prediction, fraud, network KPIs |
| [energy_utilities.md](energy_utilities.md) | filled | AMI/AMR, STLF, PMU, grid analytics, CAT modeling |
| [transportation_logistics.md](transportation_logistics.md) | filled | GPS, GTFS, AIS, route optimization, ETA prediction |
| [media_entertainment.md](media_entertainment.md) | filled | Streaming sessions, recsys, content valuation, A/B testing |
| [government_public_sector.md](government_public_sector.md) | filled | Census, admin records, weighting, geography changes |
| [agriculture.md](agriculture.md) | filled | Satellite imagery, NDVI, yield prediction, precision ag |
| [education.md](education.md) | filled | LMS, SIS, early warning systems, learning analytics |
| [insurance.md](insurance.md) | filled | GLM pricing, reserving, claims triangles, fraud detection |
| [real_estate.md](real_estate.md) | filled | AVMs, hedonic pricing, MLS, property dedup |
| [cybersecurity.md](cybersecurity.md) | filled | EDR, network flows, MITRE ATT&CK, UEBA, alert triage |

## Per-industry chapter template

Each industry chapter should follow this shape:

1. **TL;DR** — the unique data shape and constraints of this industry in 3–5 sentences.
2. **Common data types** — which of [`../data_types/`](../data_types/) appear, with concrete column names and examples.
3. **Common sources** — which of [`../data_sources/`](../data_sources/) feed this industry.
4. **Standard schemas / formats** — domain-specific schemas, e.g., FHIR (health), FIX (finance), GTFS (transit).
5. **Cleaning particulars** — beyond the generic [`../data_cleaning/`](../data_cleaning/) chapters, what this industry's data specifically requires.
6. **Standard analyses** — what data scientists in this industry are usually asked to produce (e.g., LTV, churn, risk score, anomaly detection).
7. **Standard visualizations** — domain conventions.
8. **Regulation and ethics** — PHI/HIPAA, MiFID, SOX, GDPR, PCI, COPPA, FERPA, etc.
9. **Public datasets** to learn on.
10. **References** — domain textbooks, regulatory documents, key papers.
