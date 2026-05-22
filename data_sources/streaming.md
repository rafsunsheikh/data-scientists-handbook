# Streaming Data Sources

> **TL;DR** — Streaming data arrives continuously and unbounded, unlike batch data that has a clear start and end. The fundamental challenge is processing data *as it arrives* while handling out-of-order events, late arrivals, exactly-once semantics, and backpressure. Streaming platforms (Kafka, Kinesis, Pub/Sub) act as persistent buffers between producers and consumers, enabling decoupled, fault-tolerant data pipelines. For data scientists, streaming matters for real-time features, live dashboards, anomaly detection, and as the ingestion layer for batch systems via CDC (Change Data Capture).

## 1. Streaming platforms (brokers)

### 1.1 Apache Kafka

The most widely used streaming platform. Distributed commit log with partitioned topics.

**Core concepts:**

- **Topic:** A category/stream of records. Split into partitions for parallelism.
- **Partition:** Ordered, immutable sequence of records. Each record has an offset within its partition.
- **Producer:** Writes records to topics. Can partition by key (same key → same partition).
- **Consumer:** Reads records from topics. Consumers group into consumer groups for parallelism.
- **Consumer group:** Multiple consumers share a topic; each partition goes to exactly one consumer in the group.
- **Offset:** Record position within a partition. Consumers commit offsets to track progress.
- **Retention:** Time-based (7 days default) or size-based. Records expire after retention.
- **Replication:** Partitions are replicated across brokers for fault tolerance.

**Delivery semantics:**

| Semantic | Guarantee | Tradeoff |
|---|---|---|
| **At-most-once** | Record delivered 0 or 1 time | Fastest; records can be lost |
| **At-least-once** | Record delivered 1+ times | Records can be duplicated; idempotency needed |
| **Exactly-once** | Record delivered exactly once | Requires transactional producers + idempotent consumers; slowest |

**Python:**

```python
from kafka import KafkaProducer, KafkaConsumer
import json

# Producer
producer = KafkaProducer(
    bootstrap_servers="localhost:9092",
    value_serializer=lambda v: json.dumps(v).encode("utf-8"),
    key_serializer=lambda k: k.encode("utf-8"),
)
producer.send("events", key="user123", value={"event": "click", "timestamp": "2024-01-01T00:00:00Z"})
producer.flush()

# Consumer
consumer = KafkaConsumer(
    "events",
    bootstrap_servers="localhost:9092",
    value_deserializer=lambda m: json.loads(m.decode("utf-8")),
    key_deserializer=lambda k: k.decode("utf-8"),
    group_id="my-consumer-group",
    auto_offset_reset="earliest",  # or "latest"
)
for msg in consumer:
    process(msg.key, msg.value)
    consumer.commit()  # commit offset
```

### 1.2 Amazon Kinesis

AWS managed streaming service.

**Core concepts:**

- **Stream:** Equivalent to Kafka topic.
- **Shard:** Equivalent to Kafka partition. Each shard supports 1 MB/s write, 2 MB/s read, 1000 writes/sec.
- **Records:** 1 KB max per record.
- **Retention:** 1–7 days (default 24 hours).
- **Consumer types:**
  - **Enhanced fan-out:** 1 MB/s per consumer per shard (separate from shared throughput).
  - **Shared throughput:** Up to 2 MB/s across all consumers per shard.

**What data scientists get:**

- Fully managed (no broker maintenance).
- Integrates with S3 (for batch replay), Lambda (for processing), Redshift (for analytics).
- Firehose: Auto-loads streaming data to S3, Redshift, Elasticsearch, Splunk.

### 1.3 Google Cloud Pub/Sub

Google's managed messaging service.

**Core concepts:**

- **Topic:** Named resource that records store.
- **Subscription:** Consumers pull from or are pushed subscriptions to a topic.
- **Ordering:** Per-key ordering within a topic. No global ordering guarantee.
- **Retention:** 7 days default (configurable up to 31 days).
- **At-least-once delivery:** Messages may be delivered multiple times.

**What data scientists get:**

- Global distribution (multi-region topics).
- Integrates with BigQuery (streaming inserts), Dataflow (processing), Cloud Functions.
- Simpler model than Kafka (no partitions, shards, or consumer groups to manage).

### 1.4 Azure Event Hubs

Azure's big-data streaming platform.

**Core concepts:**

- **Event hub:** Equivalent to Kafka topic.
- **Partition:** Similar to Kafka partitions.
- **Consumer groups:** Multiple independent readers.
- **Retention:** 1–90 days.
- **Kafka support:** Event Hubs supports the Kafka protocol natively.

### 1.5 Lightweight streaming

| Platform | Protocol | Use case |
|---|---|---|
| **MQTT** | MQTT (publish/subscribe) | IoT, low-bandwidth, mobile |
| **NATS** | Custom binary protocol | Low-latency microservices |
| **Redis Streams** | Redis commands | Simple streaming, in-memory |
| **Pulsar** | Custom protocol + Kafka API | Cloud-native, multi-tenancy |

**MQTT:**

- **Topics:** Hierarchical (`home/bedroom/temperature`).
- **QoS levels:** 0 (at-most-once), 1 (at-least-once), 2 (exactly-once).
- **Retained messages:** Last message stored and sent to new subscribers.
- **Last Will and Testament (LWT):** Notification when a client disconnects unexpectedly.
- **Python:** `paho-mqtt` library.

## 2. Change Data Capture (CDC)

CDC captures row-level changes in a database and streams them.

### 2.1 How CDC works

1. **Source database** writes to its transaction log (WAL in Postgres, binlog in MySQL, redo log in Oracle).
2. **CDC connector** reads the transaction log (no impact on query performance).
3. **Changes** are emitted as a stream of insert/update/delete events.
4. **Destination** (data warehouse, search index, cache) applies the changes.

### 2.2 CDC tools

| Tool | Source | Destination | Notes |
|---|---|---|---|
| **Debezium** | MySQL, Postgres, Oracle, SQL Server, MongoDB | Kafka | Open-source, most widely used |
| **AWS DMS** | Any supported source | S3, Redshift, DynamoDB | Managed, AWS-only |
| **Confluent Kafka Connect** | JDBC, Postgres, MongoDB | Kafka | Enterprise |
| **Materialize** | Kafka, Postgres | Materialized views | Real-time SQL on streaming data |
| **RisingWave** | Kafka, Postgres | Streaming SQL | Cloud-native streaming DB |

### 2.3 CDC event format

```json
{
    "op": "u",           // c=create, u=update, d=delete, r=read
    "ts_ms": 1704067200000,
    "before": {"id": 1, "name": "Alice", "email": "alice@old.com"},
    "after": {"id": 1, "name": "Alice", "email": "alice@new.com"},
    "source": {
        "db": "production",
        "table": "users",
        "txId": 42
    }
}
```

### 2.4 What data scientists get from CDC

- **Near-real-time data warehouse updates** without batch ETL.
- **Audit trails** — every change is recorded.
- **Feature store updates** — features derived from streaming data stay fresh.
- **Replay** — since events are stored in Kafka, you can reprocess history.

## 3. Schema registries

As data evolves, consumers and producers need a shared schema.

### 3.1 Schema types

| Format | Features |
|---|---|
| **Avro** | Binary, schema evolution (reader/writer schema), default values |
| **Protobuf** | Binary, compact, Google's format, strong typing |
| **JSON Schema** | Text-based, human-readable, less efficient |
| **Avro (Confluent)** | Avro with schema registry, backward/forward/complete compatibility |

### 3.2 Schema evolution rules

| Direction | Rule | Safe? |
|---|---|---|
| **Backward** | New schema can read data written by old schema | Yes (new producer, old consumer) |
| **Forward** | Old schema can read data written by new schema | Yes (old producer, new consumer) |
| **Full** | Both backward and forward compatible | Most restrictive |

**Common evolution actions:**

- **Add field with default:** Always safe.
- **Remove field:** Safe if no consumers need it.
- **Change field type:** Only if compatible (int → long, string → string).
- **Rename field:** Not natively supported — add new, deprecate old.

### 3.3 Python

```python
from confluent_kafka import Deserializer, Serializer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer, AvroSerializer

schema_registry = SchemaRegistryClient({"url": "http://schema-registry:8081"})

deserializer = AvroDeserializer(
    schema_registry_client=schema_registry,
    schema_str='{"type": "record", "name": "User", "fields": [{"name": "id", "type": "int"}, {"name": "name", "type": "string"}]}',
)

producer = {"url": "http://schema-registry:8081"})
```

## 4. Stream processing

### 4.1 Processing models

| Model | Description | Tools |
|---|---|---|
| **Micro-batch** | Process small batches (seconds) | Spark Structured Streaming |
| **True streaming** | Process record-by-record | Flink, Kafka Streams |
| **Trigger-based** | Process when data arrives or timer fires | Dataflow, Beam |

### 4.2 Windowing

Since streams are unbounded, you need windows to compute aggregates.

| Window type | Description | Use case |
|---|---|---|
| **Tumbling** | Fixed-size, non-overlapping (e.g., 1-minute) | Count events per minute |
| **Sliding** | Fixed-size, overlapping (e.g., 5-min window, 1-min advance) | Rolling averages |
| **Session** | Gap-based (close window after N seconds of inactivity) | User activity sessions |
| **Global** | No window (operate on entire stream) | Rarely used in practice |

**Python (PyFlink):**

```python
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.datastream.window import TumblingEventTimeWindows

env = StreamExecutionEnvironment.get_execution_environment()
stream = env.from_source(..., source, WatermarkStrategy.for_monotonous_timestamps())

windowed = stream.key_by(lambda x: x["user_id"]).window(TumblingEventTimeWindows.of("1 minute"))
aggregated = windowed.aggregate(MyAggregateFunction())
```

### 4.3 Watermarks and late data

Watermarks define how long the system waits for late events before closing a window.

- **Bounded out-of-orderness:** "Wait 5 seconds for late events."
- **Watermark:** System time minus 5 seconds.
- **Late data handling:** Send to a side output (dead-letter) for later processing.
- **Allowed lateness:** Re-open closed windows for late events (resource-intensive).

### 4.4 Backpressure

When consumers can't keep up with producers, data backs up in the buffer.

**Symptoms:**

- Increasing consumer lag.
- Increasing memory usage in the broker.
- Dropping records (if retention expires).

**Mitigations:**

- Add more partitions (increases parallelism).
- Add more consumers in the group.
- Speed up processing (optimize queries, add compute).
- Increase retention (buy time).
- Sample or drop low-priority events.

## 5. Stream processing tools

| Tool | Language | Model | Notes |
|---|---|---|---|
| **Apache Flink** | Java/Scala/Python | True streaming | Stateful processing, exactly-once, most powerful |
| **Spark Structured Streaming** | Scala/Python/Java | Micro-batch | Unified batch + streaming, widely used |
| **Kafka Streams** | Java/Scala | True streaming | Library (not standalone), sits in your app |
| **Faust** | Python | True streaming | Kafka-native Python streaming library |
| **Bytewax** | Python | True streaming | Modern, declarative, async |
| **Apache Beam** | Java/Python/Go | Unified | Run on Flink, Spark, Dataflow |
| **ksqlDB** | SQL | True streaming | SQL on Kafka streams |
| **Materialize** | SQL | True streaming | Streaming data warehouse |

### 5.1 Faust (Python)

```python
import faust

app = faust.App("my-app", broker="kafka://localhost:9092")
topic = app.topic("events", value_type=dict)

@app.agent(topic)
async def process(events):
    async for event in events:
        # Stateful processing
        windowed = event.window(60)  # 60-second window
        await handle(event)
```

### 5.2 Bytewax (Python)

```python
from bytewax.dataflow import Dataflow
from bytewax.operators import reduce_window, scan_window
from bytewax.windowing import TumblingWindows, WindowSchedule

class SumReducer:
    def __init__(self): pass
    def begin(self): return 0
    def __call__(self, acc, x): return acc + x

flow = Dataflow("stream-sum")
flow.input("input", InputSource("kafka://localhost:9092", "events"))
flow.map(lambda x: int(x))
flow.reduce_window(SumReducer(), TumblingWindows(schedule=WindowSchedule.seconds(60)))
flow.output("output", OutputSink("console"))
```

## 6. Offline reprocessing

One advantage of streaming platforms: data is retained, so you can replay from any offset.

**Use cases:**

- Fix a bug in the processing logic — replay last 7 days.
- Re-train a model with new features — replay raw events.
- Audit / forensics — reproduce what happened.

**Kafka replay:**

```bash
# Consume from offset 0, write to another topic
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic source-topic --from-beginning \
    | kafka-console-producer.sh --bootstrap-server localhost:9092 \
    --topic replay-topic
```

## 7. Connecting to analytical stores

Streaming data often feeds analytical databases for querying.

| Platform | Connection | Notes |
|---|---|---|
| **Materialize** | Kafka source + SQL materialized views | Sub-second queries on streaming data |
| **RisingWave** | Kafka source + streaming SQL | Cloud-native, auto-scaling |
| **ClickHouse** | Kafka engine — native Kafka consumer | Direct inserts into ClickHouse tables |
| **Delta Lake** | Structured Streaming → Delta | Lambda architecture (stream + batch on same format) |
| **Apache Iceberg** | Flink/Spark streaming → Iceberg | Streaming upserts into data lake |

## 8. Common pitfalls for data scientists

### 8.1 Assuming ordering

Records in a Kafka topic are ordered *within a partition*, not across partitions. If you need global ordering, use a single partition (bottleneck).

### 8.2 Ignoring late data

Events arrive late (network delays, clock skew). Windows close too early → counts are wrong. Set watermarks and handle late data.

### 8.3 Not idempotent consumers

At-least-once delivery means duplicates. If your consumer writes to a database, duplicates cause double-counting. Use idempotent writes (upserts with unique keys).

### 8.4 Consumer group misconfiguration

Too few consumers → bottleneck. Too many → idle consumers (wasted). One consumer per partition is optimal.

### 8.5 Clock skew

Event time vs. processing time. If producers have inconsistent clocks, event-time windows are wrong. Use processing time for simple cases, event time with watermarks for accuracy.

### 8.6 Schema drift without registry

Producers send new fields or change types without updating consumers. Always use a schema registry with evolution rules.

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `kafka-python` / `confluent-kafka` | Kafka client |
| `faust` | Python stream processing on Kafka |
| `bytewax` | Modern Python stream processing |
| `pyflink` | Apache Flink on Python |
| `pyspark` | Spark Structured Streaming |
| `apache-beam` | Unified batch/streaming SDK |
| `debezium` | CDC connector (Java, but Python can trigger) |
| `aiokafka` | Async Kafka client for asyncio |
| `paho-mqtt` | MQTT client |
| `duckdb` | SQL on streaming data (via external tables) |

## 10. References

- Kleppmann, M. *Designing Data-Intensive Applications*. O'Reilly (2017). (Ch. 6: Streaming, Ch. 7: The Data Pipeline)
- Apache Kafka Documentation — https://kafka.apache.org/documentation/
- Apache Flink Documentation — https://nightlies.apache.org/flink/flink-docs-stable/
- Debezium Documentation — https://debezium.io/documentation/
- Google Cloud Pub/Sub Documentation — https://cloud.google.com/pubsub/docs
- Amazon Kinesis Documentation — https://docs.aws.amazon.com/kinesis/
