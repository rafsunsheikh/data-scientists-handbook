# APIs as Data Sources

> **Stub — contributions welcome.**

## What to cover

- REST: status codes, pagination patterns (offset, cursor, link headers), retries.
- GraphQL: queries, schema introspection, batching with Dataloader.
- SOAP / XML: legacy enterprise; using `zeep`.
- Webhooks: idempotency, signature verification, dead-letter queues.
- Authentication: API keys, OAuth 2 flows, JWT, mutual TLS.
- Rate limiting and backoff (Retry-After, token buckets, jitter).
- Bulk export endpoints vs. live API.
- Schema evolution; versioning; deprecation.
- Robust client patterns: `httpx` + tenacity, `aiohttp`, generated clients (openapi-python-client).
- Storing raw responses (the "bronze" layer) vs. parsed.
- Cost / quota tracking.
