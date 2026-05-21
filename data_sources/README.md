# Data Sources

> **Status: stubs — looking for contributors.** Each file in this folder lists what should be covered. See [CONTRIBUTING.md](../CONTRIBUTING.md).

A data scientist's first practical question on any new project is: *where is this data going to come from?* Each source has its own ingestion mechanics, freshness guarantees, schema stability, access controls, licensing, and failure modes.

## Index

| Chapter | What it covers |
|---|---|
| [databases.md](databases.md) | Relational (Postgres, MySQL), OLAP (Snowflake, BigQuery, Redshift), NoSQL (Mongo, Cassandra), vector DBs |
| [apis.md](apis.md) | REST, GraphQL, SOAP, webhooks, pagination, rate limiting, auth (OAuth, API keys) |
| [files.md](files.md) | CSV, TSV, JSON/JSONL, Parquet, Avro, Excel, PDFs, archive formats |
| [web_scraping.md](web_scraping.md) | HTML parsing, headless browsers, robots.txt, ethics, anti-bot countermeasures |
| [streaming.md](streaming.md) | Kafka, Kinesis, Pub/Sub, MQTT, CDC, change streams |
| [sensors_iot.md](sensors_iot.md) | Industrial sensors, wearables, smart-home, edge devices |
| [surveys.md](surveys.md) | Survey design, sampling, biases, longitudinal studies |
| [public_datasets.md](public_datasets.md) | Kaggle, HuggingFace, government open data, Census, World Bank, OpenStreetMap |
| [synthetic.md](synthetic.md) | Simulation, agent-based, LLM-generated, GAN-generated, augmentation |

## Cross-cutting concerns (apply to every source)

When integrating *any* source, document at minimum:

1. **Identity** — system name, owner, contact, primary vs. secondary.
2. **Schema** — fields, types, semantics, unit. Where is it documented?
3. **Freshness** — how often updated? what's the SLA?
4. **Volume** — rows/day or events/sec.
5. **Access** — who can read it? How (creds, IP allowlist, VPN)?
6. **License / terms of use** — including data-residency constraints.
7. **PII / sensitive content** — and the policy that governs it.
8. **Failure mode** — what happens when the source is down or returns bad data?
9. **Lineage** — where does the upstream data come from?
10. **Cost** — query / egress / storage cost.
