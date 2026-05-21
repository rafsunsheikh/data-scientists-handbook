# Industries

> **Status: partially filled.** Five chapters are written in depth; the other ten are stubs looking for contributors. If you have working experience in one of the stubbed industries, please write it up. See [CONTRIBUTING.md](../CONTRIBUTING.md).

Every industry has its own data shapes, primary sources, regulatory constraints, standard analyses, and gotchas. This section is a map of *where* the abstract data-type / cleaning / visualization knowledge lands in practice.

## Index

| Industry | Status | Distinguishing data |
|---|---|---|
| [healthcare.md](healthcare.md) | filled | EHR, claims, imaging, genomics, wearables; PHI / HIPAA |
| [finance.md](finance.md) | filled | Tick / OHLCV, transactions, balances, KYC, risk models |
| [retail_ecommerce.md](retail_ecommerce.md) | filled | Orders, sessions, catalog, inventory, returns |
| [manufacturing.md](manufacturing.md) | filled | Sensor / SCADA, MES, quality, yield, supply chain |
| [marketing_advertising.md](marketing_advertising.md) | filled | Impressions, clicks, attribution, audience, A/B |
| [telecom.md](telecom.md) | stub | CDRs, network logs, NPS, churn |
| [energy_utilities.md](energy_utilities.md) | stub | Smart meter, grid sensors, weather, demand |
| [transportation_logistics.md](transportation_logistics.md) | stub | GPS, AIS, shipment tracking, routing, capacity |
| [media_entertainment.md](media_entertainment.md) | stub | Streaming, recommendations, content metadata, ads |
| [government_public_sector.md](government_public_sector.md) | stub | Census, surveys, administrative records, open data |
| [agriculture.md](agriculture.md) | stub | Satellite, drone, sensors, yield, soil, weather |
| [education.md](education.md) | stub | LMS, assessments, attendance, outcomes |
| [insurance.md](insurance.md) | stub | Policies, claims, underwriting, fraud |
| [real_estate.md](real_estate.md) | stub | Listings, transactions, geospatial, valuation |
| [cybersecurity.md](cybersecurity.md) | stub | Logs, network flow, endpoint events, threat intel |

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
