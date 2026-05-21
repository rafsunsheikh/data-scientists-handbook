# Databases as Data Sources

> **Stub — contributions welcome.**

## What to cover

- Relational (OLTP): Postgres, MySQL, SQL Server, Oracle — strengths, dialect quirks, JSONB / arrays.
- Analytical (OLAP / warehouse): Snowflake, BigQuery, Redshift, Databricks SQL, ClickHouse, DuckDB.
- NoSQL: MongoDB, Cassandra, DynamoDB, Couchbase — document, wide-column, key-value.
- Graph DBs: Neo4j, Memgraph, ArangoDB, TigerGraph.
- Vector DBs: pgvector, Pinecone, Weaviate, Milvus, Qdrant, Chroma.
- Time-series DBs: InfluxDB, TimescaleDB, QuestDB.
- Connection patterns: SQLAlchemy, raw DB-API drivers, dbt, polars-sql.
- Query strategy: pushdown vs. pull-down, sampling, partitioned reads.
- Replicas and CDC; reading from a primary in production is usually wrong.
- Cost optimization (warehouse query cost, indexes, materialized views).
