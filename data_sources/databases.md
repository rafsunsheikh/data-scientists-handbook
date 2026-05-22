# Databases as Data Sources

> **TL;DR** — Databases are the most common data source for data scientists, but the type of database fundamentally shapes what you can query, how fast you can get answers, and what assumptions you can make about data consistency. Relational databases give you ACID guarantees and a mature query ecosystem but struggle with unstructured or hierarchical data. NoSQL databases trade consistency for scale and flexibility, each sub-type optimized for a different data shape. Knowing which database to query and how its internals affect your queries is half the battle.

## 1. Sub-types and when they apply

### 1.1 Relational databases (RDBMS)

Store data in structured tables with rows and columns, enforce schema and relationships via foreign keys.

| Database | Language | Typical DS use |
|---|---|---|
| **PostgreSQL** | SQL | General-purpose analytics, JSONB for semi-structured, extensions (PostGIS, pgvector) |
| **MySQL / MariaDB** | SQL | Web application data, operational analytics |
| **SQLite** | SQL | Embedded, prototyping, small datasets, mobile apps |
| **Oracle** | SQL | Enterprise legacy, financial systems |
| **SQL Server** | SQL | Microsoft ecosystem, Power BI integration |
| **Postgres + Citus** | SQL | Distributed Postgres for larger workloads |

**Key parameters:**

- **Schema:** Fixed, defined DDL. Columns have types (integer, text, timestamp, boolean, array, JSON).
- **Connection model:** Persistent TCP connections, connection pooling essential at scale.
- **Query language:** SQL (SELECT, JOIN, GROUP BY, window functions, CTEs).
- **Transaction isolation:** Read Committed, Repeatable Read, Serializable (varies by DB).
- **ACID:** Atomicity, Consistency, Isolation, Durability — guaranteed.

**What data scientists get:**

- Reliable joins across multiple tables.
- Window functions for time-series and ranking.
- Materialized views for pre-computed aggregates.
- Export to CSV/Parquet via `COPY`, `SELECT INTO`, or `pg_dump`.

### 1.2 OLAP / columnar databases

Optimized for read-heavy analytical queries over large datasets. Data is stored by column, not row, enabling fast aggregation on subsets of columns.

| Database | Language | Typical DS use |
|---|---|---|
| **Snowflake** | SQL | Cloud data warehouse, elastic compute/storage |
| **BigQuery** | SQL | Serverless warehouse, built-in ML, geospatial |
| **Redshift** | SQL | AWS data warehouse, sort/key indexes |
| **Databricks (Delta Lake)** | SQL + Spark | Lakehouse, open format, multi-engine |
| **ClickHouse** | SQL | Real-time analytics, massive row counts |
| **DuckDB** | SQL | In-process analytical queries on local files |
| **Greenplum** | SQL | MPP (massively parallel processing) warehouse |
| **StarRocks / Apache Doris** | SQL | Sub-second analytics on large datasets |

**Key parameters:**

- **Storage:** Columnar (data for each column stored contiguously).
- **Compute:** Often decoupled from storage (Snowflake, BigQuery) or shared (Redshift).
- **Query language:** SQL (dialect varies). Most support standard SQL with extensions.
- **Partitioning:** By date, hash, or range. Critical for query performance.
- **Clustering / sort keys:** Determines physical ordering within partitions.

**What data scientists get:**

- Petabyte-scale queries without moving data.
- Built-in ML (BigQuery ML, Databricks MLflow, Redshift ML).
- Time-travel / versioning (Delta Lake, Snowflake Time Travel, BigQuery).
- Easy integration with Python via connectors or `dbt`.

### 1.3 Document databases

Store data as documents (typically JSON/BSON). Schema is flexible — each document can have different fields.

| Database | Format | Typical DS use |
|---|---|---|
| **MongoDB** | BSON | User profiles, product catalogs, content management |
| **Couchbase** | JSON | Mobile/web app data, low-latency access |
| **Firestore / Firebase** | JSON | Mobile/web app event data, real-time sync |
| **RethinkDB** | JSON | Real-time feeds, change notifications |

**Key parameters:**

- **Schema:** Flexible, schema-less. Documents in the same collection can have different fields.
- **Storage:** BSON (MongoDB) or JSON. Nested structures supported.
- **Query:** Rich query language (filter, sort, aggregate pipeline). MongoDB aggregation pipeline is MapReduce-like.
- **Indexing:** Single-field, compound, geospatial, text, hashed, TTL.
- **Replication:** Replica sets (automatic failover). Sharding for horizontal scale.

**What data scientists get:**

- Raw event data from web/mobile apps in its native nested format.
- No need to denormalize hierarchical data.
- Aggregation pipelines for ETL-like transformations.
- Export via `mongoexport`, `mongodump`, or connector.

**Pitfalls:**

- No joins (use manual `$lookup` in MongoDB — slow).
- Data quality varies — inconsistent schemas across documents.
- Querying nested arrays can be expensive.

### 1.4 Graph databases

Store data as nodes (entities) and edges (relationships). Optimized for traversing relationships.

| Database | Query language | Typical DS use |
|---|---|---|
| **Neo4j** | Cypher | Recommendation engines, fraud detection, knowledge graphs |
| **Amazon Neptune** | SPARQL / Gremlin | Knowledge graphs, relationship analytics |
| **JanusGraph** | Gremlin | Large-scale graph analytics |
| **TigerGraph** | GSQL | Real-time graph analytics |
| **ArangoDB** | AQL | Multi-model (graph + document) |

**Key parameters:**

- **Data model:** Nodes with properties, edges with properties and direction.
- **Query:** Graph traversal (Cypher: `MATCH (a)-[:FOLLOWS]->(b)`, Gremlin: `g.V().out('follows')`).
- **Indexing:** Index on node/edge properties. Traversal is O(1) after index lookup (pointer-based).
- **ACID:** Yes (Neo4j, JanusGraph).

**What data scientists get:**

- Direct access to relationship structures (friends-of-friends, supply chains, network flows).
- Graph algorithms via libraries (PageRank, community detection, shortest path).
- Feature engineering: degree centrality, betweenness, clustering coefficient, node embeddings.

**Pitfalls:**

- Not suitable for aggregations over entire tables.
- Export is harder than row-based databases.
- Schema design for graphs is non-intuitive for tabular thinkers.

### 1.5 Vector databases

Store high-dimensional vectors (embeddings) optimized for similarity search (nearest neighbor).

| Database | Query type | Typical DS use |
|---|---|---|
| **Pinecone** | ANN search | Semantic search, recommendation, RAG |
| **Milvus** | ANN search | Large-scale vector search, open-source |
| **Weaviate** | ANN search + CRUD | Vector search with schema, hybrid search |
| **Qdrant** | ANN search | Fast filtering + vector search |
| **Faiss (Facebook)** | ANN search | CPU/GPU vector similarity, library not server |
| **Annoy (Spotify)** | ANN search | Static vector index, memory-mapped |
| **pgvector** | ANN search | Vector search inside PostgreSQL |
| **Chroma** | ANN search | Embedding storage for LLM apps |

**Key parameters:**

- **Vector dimension:** Fixed per collection (e.g., 1536 for OpenAI ada-002, 384 for sentence-transformers).
- **Index type:**
  - **HNSW** (Hierarchical Navigable Small World) — fast, memory-heavy, best for most use cases.
  - **IVF** (Inverted File Index) — clustering-based, memory-efficient.
  - **DiskANN** — disk-based for very large datasets.
  - **Flat / brute-force** — exact search, slow at scale.
- **Distance metric:** Cosine (most common for embeddings), L2/Euclidean, inner product, Hamming (binary).
- **Consistency:** Approximate (not exact) nearest neighbor — trade precision for speed.

**What data scientists get:**

- Embedding similarity search (semantic, image, audio).
- RAG (Retrieval-Augmented Generation) pipelines.
- Recommendation via item-item or user-item vector similarity.
- Anomaly detection via distance from cluster center.

**Pitfalls:**

- Curse of dimensionality — similarity becomes meaningless at very high dimensions without proper normalization.
- No relational joins — vectors live in isolation.
- Index rebuilds can be expensive at scale.
- Query results are approximate — recall@k is the key metric, not exactness.

### 1.6 Time-series databases

Optimized for ingesting and querying time-stamped data at high write throughput.

| Database | Query language | Typical DS use |
|---|---|---|
| **InfluxDB** | Flux / InfluxQL | IoT, monitoring, observability |
| **TimescaleDB** | SQL (PostgreSQL extension) | Application time-series, SQL analytics |
| **TDengine** | SQL-like | IoT, industrial data |
| **QuestDB** | SQL + ILP | Real-time analytics on streaming data |
| **Prometheus** | PromQL | Metrics / monitoring (not general DS) |
| **DynamoDB** (with time key) | Key-value | High-throughput time-series at scale |

**Key parameters:**

- **Schema:** Time is always the primary key. Often partitioned by time.
- **Write throughput:** Designed for millions of writes/sec per node.
- **Compression:** Delta-of-delta encoding, GORILLA float compression. Data is highly compressible.
- **Retention policies:** Automatic data expiration (TTL). Downsampling to older, coarser granularity.
- **Query:** Time-range filters, downsampling (GROUP BY time interval), gap-filling.

**What data scientists get:**

- Native time-series operations (rolling windows, resampling, forward-fill).
- High-ingest rate for sensor / telemetry data.
- Efficient storage via compression.
- Often SQL-compatible (TimescaleDB, QuestDB) for familiar querying.

**Pitfalls:**

- Limited join support.
- Not designed for ad-hoc schema changes.
- Export for offline analysis can be slow (designed for streaming reads).

### 1.7 Key-value / wide-column databases

| Database | Model | Typical DS use |
|---|---|---|
| **Redis** | Key-value | Caching, real-time features, session data |
| **Cassandra / ScyllaDB** | Wide-column | Write-heavy time-series, event logging |
| **HBase** | Wide-column | Large-scale tabular data on HDFS |
| **RocksDB** | Embedded key-value | Local feature store, low-latency lookup |

**What data scientists get:**

- Real-time feature lookup (feature stores often sit on Redis / RocksDB).
- High-throughput event ingestion (Cassandra).
- Session / cache data for A/B testing.

**Pitfalls:**

- No SQL — query language is limited to key-based lookups or secondary indexes.
- Export is non-standard.
- Schema design is tightly coupled to access patterns.

## 2. Cross-cutting concerns

### 2.1 Connection and access

| Method | Tools | Notes |
|---|---|---|
| **Native drivers** | `psycopg2` (Postgres), `mysql-connector` (MySQL), `pymongo` (MongoDB), `neo4j` (Neo4j) | Direct TCP connection, most control |
| **ODBC / JDBC** | `pyodbc`, `jaydebeapi` | Universal but slower |
| **ORM** | SQLAlchemy, Peewee, Django ORM | Abstraction layer, can generate inefficient queries |
| **SQLAlchemy** | `sqlalchemy.create_engine()` | Python standard for DB abstraction; works with pandas `read_sql` |
| **dbt** | dbt CLI / dbt Cloud | SQL-based transformation layer on top of warehouses |
| **Cloud connectors** | `google-cloud-bigquery`, `boto3` (Redshift/S3) | SDK-style access |

### 2.2 Query patterns that matter for data science

- **Window functions** — `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`, `SUM() OVER (PARTITION BY ... ORDER BY ...)`. Available in RDBMS and OLAP. Not in NoSQL.
- **CTEs (Common Table Expressions)** — `WITH` clauses for readable multi-step queries. Standard SQL.
- **Materialized views** — Pre-computed query results that refresh periodically. Critical for slow analytical queries.
- **Sampling** — `TABLESAMPLE` (Postgres), `SAMPLE` (BigQuery). Essential for prototyping on large datasets.
- **Export** — `COPY TO` (Postgres), `bq extract` (BigQuery), `s3distcp` (Redshift/S3), `mongodump`.

### 2.3 Data consistency models

| Model | Guarantee | Databases | DS implication |
|---|---|---|---|
| **ACID** | Atomicity, Consistency, Isolation, Durability | Postgres, MySQL, Oracle, SQL Server | Reliable transactions; data won't be in inconsistent state |
| **BASE** | Basically Available, Soft state, Eventual consistency | MongoDB, Cassandra, DynamoDB | Reads may return stale data; important for real-time analytics |
| **Eventual consistency** | Reads eventually converge | All NoSQL | Window of inconsistency after writes — factor into time-bound analysis |

### 2.4 Schema evolution

- **RDBMS:** ALTER TABLE is expensive on large tables. Use migration tools (Flyway, Liquibase). Add columns, not rename.
- **Document DBs:** Schema-free by design. But changing query patterns requires re-indexing.
- **Columnar warehouses:** Schema changes require table rebuilds (costly at petabyte scale).
- **Schema-on-read:** Data lakes (Parquet on S3) have no schema at write time. Schema applied at read. Flexible but error-prone.

### 2.5 Security and access control

- **Authentication:** Username/password, OAuth, IAM roles (cloud), certificate-based.
- **Authorization:** Row-level security (Postgres), column-level (BigQuery), attribute-based (ABAC).
- **Encryption:** At rest (AES-256), in transit (TLS).
- **Auditing:** Query logs, access logs. Required for compliance (HIPAA, SOC 2).
- **Masking / tokenization:** PII redaction at query time (BigQuery Data Masking, Azure Dynamic Data Masking).

## 3. Common pitfalls for data scientists

### 3.1 N+1 query problem

Loading a parent record, then issuing a query per child row. Common when using ORMs. Fix: use JOINs or `IN` clauses.

### 3.2 SELECT * in production queries

Wastes bandwidth, memory, and time. Always select only what you need. Columnar databases make this especially impactful.

### 3.3 Implicit data conversion

Comparing a string column to an integer, or a timestamp without timezone to one with timezone. The database may cast implicitly, using the wrong index, or produce wrong results.

### 3.4 Cartesian products

Missing `ON` or `WHERE` clause in a JOIN produces a cross join. On large tables, this hangs the query or OOMs the warehouse.

### 3.5 Timezone naivety

Timestamps stored as strings or without timezone info. Converting between UTC and local time without explicit handling. Always store in UTC.

### 3.6 Copying data out of production

Running `SELECT * FROM huge_table` on a production database can bring it down. Always:
- Use read replicas.
- Sample or filter.
- Export in batches.
- Coordinate with DBAs.

### 3.7 Assuming referential integrity in NoSQL

Document and key-value databases don't enforce foreign keys. Orphaned records are common. Validate relationships in your query logic.

## 4. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `SQLAlchemy` | Python SQL toolkit and ORM |
| `psycopg2` / `psycopg3` | PostgreSQL driver |
| `pymysql` | MySQL driver |
| `pymongo` | MongoDB driver |
| `neo4j` | Neo4j graph driver |
| `duckdb` | In-process SQL OLAP |
| `great_expectations` | Data validation / profiling |
| `pandera` | DataFrame type checking |
| `dbt` | SQL transformation layer |
| `SQLFluff` | SQL linter |
| `DBeaver` / `DataGrip` / `pgAdmin` | GUI database clients |
| `apache-airflow` | Query orchestration / scheduling |

## 5. References

- Connolly, T. & Begg, C. *Database Systems: A Practical Approach to Design, Implementation, and Management*.
- Kleppmann, M. *Designing Data-Intensive Applications*.
- PostgreSQL Documentation — https://www.postgresql.org/docs/
- MongoDB Documentation — https://www.mongodb.com/docs/
- Google BigQuery Documentation — https://cloud.google.com/bigquery/docs
- Snowflake Documentation — https://docs.snowflake.com/
- Neo4j Documentation — https://neo4j.com/docs/
- Abadi, D. & Marcus, A. *NoSQL: A Family that Thinks Like a Family*. SIGMOD Record (2010).
