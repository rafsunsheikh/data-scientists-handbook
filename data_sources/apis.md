# APIs as Data Sources

> **TL;DR** — APIs (Application Programming Interfaces) are the primary way data flows between services in modern tech. REST APIs dominate, but GraphQL, SOAP, and webhooks each solve different problems. The core challenges are pagination at scale, rate limiting under load, authentication complexity, schema drift across versions, and the gap between the "happy path" documented in examples and the messy reality of production data. Knowing how to build a robust API client — with retries, backoff, error handling, and raw-response logging — is a foundational skill for data scientists who pull their own data.

## 1. API types and protocols

### 1.1 REST APIs

Representational State Transfer APIs expose resources as URLs with standard HTTP methods.

**Core concepts:**

- **Resources:** Nouns, not verbs. `/users/123/orders` not `getOrdersForUser/123`.
- **HTTP methods:** `GET` (read), `POST` (create), `PUT` (replace), `PATCH` (partial update), `DELETE`.
- **Status codes:**
  - `200 OK` — success
  - `201 Created` — resource created
  - `204 No Content` — success, no body
  - `400 Bad Request` — malformed input
  - `401 Unauthorized` — missing / invalid auth
  - `403 Forbidden` — authenticated but not allowed
  - `404 Not Found` — resource doesn't exist
  - `429 Too Many Requests` — rate limited
  - `500 Internal Server Error` — server crash
  - `502/503/504` — gateway / upstream failures
- **Request/response format:** Typically JSON. Sometimes form-encoded (`application/x-www-form-urlencoded`) or multipart.
- **Idempotency:** `GET`, `PUT`, `DELETE` are idempotent (repeating has same effect). `POST` is not.

**Pagination patterns:**

| Pattern | How it works | Example |
|---|---|---|
| **Offset-based** | `?offset=0&limit=100` | Simple but breaks with inserts during pagination |
| **Cursor-based** | `?cursor=abc123&limit=100` | Safe for real-time data; cursor encodes position |
| **Page-based** | `?page=1&per_page=100` | Common in GitHub, Stripe |
| **Link headers** | `Rel: next` in `Link:` header | GitHub API, Shopify |
| **Keyset / seek** | `?after_id=12345&limit=100` | Better than offset for large datasets |

**Pagination pitfalls:**

- **Missing records:** With offset pagination, rows inserted between paginated requests are skipped.
- **Duplicate records:** Rows deleted between requests cause duplicates.
- **No total count:** Cursor-based pagination rarely provides a total count. You may not know when you've reached the end.
- **Hidden pagination:** Some APIs paginate internally on nested resources (e.g., `/users/123/posts` but each post has comments paginated separately).

**Query parameters:**

- Filtering: `?status=active&created_after=2024-01-01`
- Sorting: `?sort=-created_at` (neg prefix = descending, varies by API)
- Field selection: `?fields=id,name,email` (GraphQL-style, in some REST APIs)
- Expansion / nesting: `?expand=orders,items` (include related resources in one call)

### 1.2 GraphQL

GraphQL lets clients specify exactly what data they need in a single request.

**Core concepts:**

- **Query:** `query { user(id: "123") { name email orders { total } } }`
- **Schema:** Strongly typed, introspectable via `_schema` query.
- **Resolvers:** Server-side functions that fetch each field. Can be slow (N+1 on the server).
- **Batching:** Clients can use `DataLoader` (concept) to batch field resolutions.
- **Mutations:** Write operations (`mutation { createUser(...) { ... } }`).
- **Subscriptions:** Real-time via WebSocket (`subscription { onOrderCreated { ... } }`).

**Advantages over REST:**

- No over-fetching (request only the fields you need).
- No under-fetching (get nested data in one request).
- Schema is self-documenting.
- Strong typing helps client code generation.

**Disadvantages:**

- Complex error handling (partial errors — some fields succeed, some fail).
- Caching is harder (not HTTP-cacheable by default; needs Apollo, Relay, or custom cache).
- Server complexity — resolver N+1 problems, query depth limits.
- Rate limiting is per-query-complexity, not per-request.

**Pagination in GraphQL:**

- Cursor-based with `pageInfo { hasNextPage, endCursor }` (Relay spec).
- Always use cursors, never offsets.

### 1.3 SOAP / XML APIs

Simple Object Access Protocol — XML-based, enterprise legacy.

**Characteristics:**

- **WSDL:** Web Services Description Language — formal API contract (more complete than OpenAPI/Swagger).
- **XML envelopes:** Request/response wrapped in XML with SOAP headers (auth, transaction, etc.).
- **Transport:** HTTP, SMTP, TCP — not tied to HTTP.
- **Built-in features:** WS-Security, WS-ReliableMessaging, WS-AtomicTransaction.
- **Status:** Being replaced by REST/GraphQL but still in banking, telecom, government.

**What data scientists get:**

- Often the *only* way to access legacy enterprise data.
- XML parsing is slower and more verbose than JSON.
- Use `zeep` (Python) or `lxml` for parsing.
- Watch for namespaces in XML — they break naive XPath queries.

### 1.4 Webhooks

Webhooks are server-to-server callbacks — the API calls *your* endpoint when an event happens.

**Core concepts:**

- **Event-driven:** Push model (vs. API polling, which is pull).
- **Payload:** Usually JSON in an HTTP POST to your URL.
- **Signature verification:** HMAC signature in header (e.g., `X-Hub-Signature-256`) to verify the request came from the provider.
- **Idempotency:** Webhooks can be delivered multiple times. Use event IDs for idempotent processing.
- **Retry policy:** Providers retry on failure (exponential backoff). Your endpoint must be resilient.
- **Dead-letter queue:** Events that fail after max retries should go to a DLQ for inspection.

**Webhook pitfalls:**

- **Your endpoint must be publicly reachable** (or use tunneling like ngrok for local dev).
- **Timeout:** Most providers expect a response within 5–30 seconds. Process asynchronously (queue the event).
- **Secret rotation:** Webhook secrets rotate. Handle multiple active signatures during rotation.
- **Event ordering:** Webhooks may arrive out of order. Use timestamps or sequence numbers.

### 1.5 gRPC

gRPC is Google's high-performance RPC framework using Protocol Buffers.

**Characteristics:**

- **Protocol:** HTTP/2 (multiplexed, binary framing).
- **Serialization:** Protocol Buffers (binary, schema-defined, smaller than JSON).
- **Service definition:** `.proto` files define services, methods, messages.
- **Streaming:** Unary, server-streaming, client-streaming, bidirectional streaming.
- **Use cases:** Internal microservices, mobile backends, real-time data pipelines.

**What data scientists get:**

- Faster data transfer than REST/JSON (binary, HTTP/2 multiplexing).
- Strongly typed schemas — code generation in Python, Go, etc.
- gRPC streaming for real-time data feeds.
- Python: `grpcio` + `grpcio-tools` for client generation.

## 2. Authentication and authorization

### 2.1 Methods

| Method | Use case | Security |
|---|---|---|
| **API key** | Simplest; in header or query | Low — treat like a password; no expiration by default |
| **Bearer token (JWT)** | Stateless auth; in `Authorization: Bearer <token>` header | Medium — self-contained but can't be revoked until expiry |
| **OAuth 2.0** | Delegated access (user grants your app access to their data) | High — short-lived access tokens + refresh tokens |
| **OAuth 2.0 + PKCE** | Mobile / SPA apps (prevents auth code interception) | High — recommended for all public clients |
| **mTLS (mutual TLS)** | Service-to-service; both sides present certificates | Very high — certificates exchanged on TLS handshake |
| **HMAC signature** | Webhook verification; request signing | High — proves request integrity and source |

### 2.2 OAuth 2.0 flows

| Flow | Client type |
|---|---|
| **Authorization Code** | Server-side (backend can keep secret) |
| **Authorization Code + PKCE** | Public clients (mobile, SPA, CLI) |
| **Client Credentials** | Machine-to-machine (no user involved) |
| **Device Authorization** | Devices with limited input (smart TV, CLI) |

**What data scientists get:**

- Most third-party APIs (Google, Facebook, Salesforce) use OAuth 2.0.
- You'll need to register an app, get client ID + secret, and implement the flow.
- Tokens expire — implement refresh logic.
- Scope limits what you can access. Request only what you need.

## 3. Rate limiting and throttling

### 3.1 How rate limits work

| Header | Meaning |
|---|---|
| `RateLimit-Limit` | Max requests per window |
| `RateLimit-Remaining` | Requests left in current window |
| `RateLimit-Reset` | Unix timestamp when window resets |
| `Retry-After` | Seconds to wait (on 429 response) |
| `X-RateLimit-Reset` | Alternative: Unix timestamp |

### 3.2 Backoff strategies

| Strategy | When to use |
|---|---|
| **Fixed delay** | Simple prototyping; not recommended for production |
| **Exponential backoff** | Standard — wait 1s, 2s, 4s, 8s... |
| **Exponential backoff + jitter** | Production — add random jitter to prevent thundering herd |
| **Token bucket** | Smooth throughput; maintain a bucket of tokens refilled at a rate |
| **Adaptive** | Adjust based on observed `Retry-After` headers |

**Python: `tenacity` library**

```python
from tenacity import retry, stop_after_attempt, wait_exponential_jitter

@retry(stop=stop_after_attempt(5), wait=wait_exponential_jitter(initial=1, max=60, jitter=5))
def fetch_with_backoff(url, headers):
    resp = requests.get(url, headers=headers)
    resp.raise_for_status()
    return resp.json()
```

### 3.3 Burst vs. sustained limits

- **Burst limit:** Max requests in a short window (e.g., 100 in 1 second).
- **Sustained limit:** Max requests over a longer window (e.g., 1000 per minute).
- Design your client to respect both — throttle per-second bursts and per-minute totals.

## 4. Schema evolution and versioning

### 4.1 How APIs change

- **Additive changes:** New optional fields — usually backward compatible.
- **Removal:** Deprecated fields removed — breaks consumers.
- **Type change:** Integer → string — breaks parsing.
- **Enum expansion:** New values in a field — your code may not handle them.
- **Behavior change:** Same input, different output (e.g., rounding change).

### 4.2 Versioning strategies

| Strategy | Example | Pros | Cons |
|---|---|---|---|
| **URL path** | `/v1/users`, `/v2/users` | Explicit, easy to cache | Multiple versions maintained |
| **Query param** | `/users?version=1` | Flexible | Easy to accidentally hit wrong version |
| **Header** | `API-Version: 2024-01-01` | Clean URLs | Harder to debug; not cache-friendly |
| **Content negotiation** | `Accept: application/vnd.api+v2+json` | Standard HTTP | Complex |

### 4.3 Deprecation

- APIs announce deprecation with sunset dates.
- Monitor deprecation headers (`Sunset`, `Deprecation`).
- Subscribe to API changelogs / release notes.
- Test against new versions before old ones shut down.

## 5. Building a robust API client

### 5.1 Key patterns

```python
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential_jitter

class APIClient:
    def __init__(self, base_url, api_key, rate_limit=None):
        self.client = httpx.Client(
            base_url=base_url,
            headers={"Authorization": f"Bearer {api_key}"},
            limits=httpx.Limits(max_connections=10, max_keepalive_connections=5),
        )
        self.rate_limit = rate_limit  # requests per second

    @retry(stop=stop_after_attempt(5), wait=wait_exponential_jitter(1, 60, 10))
    def get(self, path, params=None):
        resp = self.client.get(path, params=params)

        # Handle 429 with Retry-After
        if resp.status_code == 429:
            retry_after = resp.headers.get("Retry-After", "1")
            time.sleep(float(retry_after))
            return self.get(path, params)  # retry manually

        resp.raise_for_status()
        return resp.json()

    def paginate(self, path, params=None):
        """Yield items across all pages (cursor-based)."""
        while True:
            data = self.get(path, params)
            yield from data["items"]
            if "next_cursor" not in data or not data.get("next_cursor"):
                break
            params = {**params, "cursor": data["next_cursor"]}
```

### 5.2 Best practices

1. **Always log raw responses** (the "bronze" layer) before parsing. You'll need them for debugging and re-parsing when schemas change.
2. **Use `httpx` over `requests`** for async support and HTTP/2.
3. **Implement structured logging** with request IDs for tracing.
4. **Track quota usage** — log remaining rate limit and total tokens consumed.
5. **Use a session** (connection pooling) for multiple requests.
6. **Set timeouts** — never leave them infinite. Default: 10s connect, 30s read.
7. **Handle partial errors** — some APIs return 200 with an `errors` array in the body.

## 6. Bulk export vs. live API

| Dimension | Live API | Bulk export |
|---|---|---|
| **Freshness** | Near real-time | Batch (daily/hourly) |
| **Rate limits** | Strict (10–1000 req/min) | None or very generous |
| **Schema** | Evolving, documented | Stable, tabular |
| **Use case** | Operational, real-time | Analytics, ML training |
| **Format** | JSON per resource | CSV, Parquet, Avro |
| **Delivery** | HTTPS | S3, GCS, direct download |

**Strategy:** For ML training and analytics, prefer bulk exports when available. Use the live API for fresh data or incremental updates.

## 7. Common pitfalls for data scientists

### 7.1 Not handling pagination fully

Assuming one API call gets all the data. Always verify row count matches expectations.

### 7.2 Parsing errors silently ignored

An API returns `{"error": "rate limited"}` with status 200. Always check the status code *and* the body.

### 7.3 Storing parsed instead of raw data

When the schema changes, you can't re-parse. Always store the raw JSON response.

### 7.4 Ignoring field deprecation

A field you depend on gets deprecated and removed. Use schema monitoring and field-versioning.

### 7.5 Over-polling

Polling an API every minute when a webhook or CDC stream would be cheaper and fresher.

### 7.6 Mixing environments

Testing against the production API instead of sandbox/staging. Most APIs provide separate environments.

## 8. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `httpx` | Modern HTTP client (async + sync, HTTP/2) |
| `requests` | Standard HTTP client (sync only) |
| `aiohttp` | Async HTTP client |
| `tenacity` | Retry with backoff |
| `graphene` / `gql` | GraphQL client |
| `zeep` | SOAP / XML client |
| `openapi-python-client` | Generate clients from OpenAPI specs |
| `bravado` | Swagger/OpenAPI client generator |
| `responses` / `vcrpy` | HTTP mocking for tests |
| `postman` / `insomnia` | API exploration / testing |

## 9. References

- REST: Fielding, R. T. *Architectural Styles and the Design of Network-based Software Architectures* (PhD Thesis, 2000).
- GraphQL: https://graphql.org/learn/
- OAuth 2.0: RFC 6749 — https://datatracker.ietf.org/doc/html/rfc6749
- OpenAPI Specification: https://spec.openapis.org/oas/v3.1.0
- Google API Design Guide: https://cloud.google.com/apis/design
- Stripe API Reference: https://stripe.com/docs/api (excellent API design example)
