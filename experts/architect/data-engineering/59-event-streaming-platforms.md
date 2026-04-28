# Event Streaming Platforms

## Profile

### What It Is
Event Streaming Platforms provide infrastructure for publishing, storing, and processing continuous streams of events in real-time. Unlike traditional message brokers, streaming platforms retain events in a durable, ordered log, enabling replay, stream processing, and building event-driven architectures at scale.

### Origin and Evolution
- Apache Kafka (LinkedIn, 2011) — log-based distributed streaming platform
- Apache Pulsar (Yahoo, 2016) — multi-tenant, geo-replicated streaming
- Redpanda (2020) — Kafka API-compatible, C++ implementation for performance
- Managed services: Confluent Cloud, Amazon MSK, Azure Event Hubs
- Current: Kafka Connect for integration, ksqlDB/Flink for stream processing

### Key Authors and References
- **Jay Kreps, Neha Narkhede, Jun Rao** — Kafka creators
- **Ben Stopford** — *Designing Event-Driven Systems*
- **Martin Kleppmann** — Stream processing in *DDIA*
- **Confluent** — Kafka ecosystem documentation

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Topic | Named stream of events (like a table of events) |
| Partition | Ordered subset of a topic for parallelism |
| Consumer Group | Set of consumers that share processing of a topic |
| Offset | Position of a consumer in a partition |
| Compacted Topic | Retains only the latest value per key (like a table) |
| Stream Processing | Real-time computation on event streams (Flink, ksqlDB) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Events as the source of truth** — The event log is the canonical data store. Databases and caches are derived views.
2. **Ordered, durable, replayable** — Events are stored in order, durably replicated, and can be replayed from any point.
3. **Decouple producers from consumers** — Producers write to topics. Consumers read at their own pace. They don't know about each other.
4. **Stream processing for real-time** — Transform, aggregate, and join streams in real-time for immediate insights and actions.
5. **Schema evolution** — Events will change over time. Use Schema Registry to manage backward/forward compatibility.

### When to Use vs. When to Avoid

**Use streaming platforms when:**
- High-throughput event processing (100K+ events/sec)
- Event replay and reprocessing are needed
- Multiple consumers need the same events independently
- Real-time analytics or stream processing required

**Use simpler messaging when:**
- Low throughput task queues
- Simple request-reply patterns
- No need for event replay or durability

---

## Frameworks and Methodologies

### 1. Kafka Architecture Design — step-by-step
1. Define topics by domain/event type (orders, payments, inventory)
2. Choose partition key for ordering guarantee (order_id, customer_id)
3. Set partition count based on consumer parallelism needs
4. Configure replication factor (3 for production)
5. Set retention policy (time-based, size-based, or compaction)
6. Deploy Schema Registry for event schema management

### 2. Stream Processing Pipeline — step-by-step
1. Define input streams (Kafka topics)
2. Design processing: filter, map, aggregate, join, window
3. Choose engine: Kafka Streams (library), Flink (framework), ksqlDB (SQL)
4. Implement exactly-once semantics where needed
5. Define output: new topic, database, API
6. Monitor consumer lag and processing latency

---

## How to Apply

### Decision Checklist
- [ ] Are topics designed around domain events (not technical concerns)?
- [ ] Are partition keys high-cardinality and query-aligned?
- [ ] Is Schema Registry configured for schema evolution?
- [ ] Are consumer groups designed for independent scaling?
- [ ] Is consumer lag monitored and alerted?

### Implementation Patterns

**Stream Processing with Flink SQL:**
```sql
-- Real-time order analytics
CREATE TABLE order_events (
    order_id STRING,
    customer_id STRING,
    amount DECIMAL(10,2),
    event_time TIMESTAMP(3),
    WATERMARK FOR event_time AS event_time - INTERVAL '5' SECOND
) WITH ('connector' = 'kafka', 'topic' = 'orders', ...);

-- Tumbling window aggregation
SELECT
    TUMBLE_START(event_time, INTERVAL '1' HOUR) AS window_start,
    COUNT(*) AS order_count,
    SUM(amount) AS total_revenue
FROM order_events
GROUP BY TUMBLE(event_time, INTERVAL '1' HOUR);
```

**Kafka Connect for CDC:**
```json
{
  "name": "postgres-source",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.dbname": "orders",
    "table.include.list": "public.orders,public.order_items",
    "topic.prefix": "cdc",
    "slot.name": "debezium"
  }
}
```

### Common Mistakes
1. **Too many partitions** — Over-partitioning increases overhead; start with estimated consumer count
2. **Wrong partition key** — Low cardinality keys create hot partitions
3. **No schema management** — Breaking changes in event schemas crash consumers
4. **Ignoring consumer lag** — Consumers falling behind silently; stale data without alerting
5. **Treating Kafka as a database** — Kafka stores events; don't query it like a relational database

### Integration with Other Topics
- **Event-Driven Architecture** — Streaming platforms are the backbone of EDA
- **Event Sourcing** — Kafka as event store for event-sourced systems
- **CQRS** — Events from streaming platforms update read models
- **Data Pipelines** — Streaming as alternative to batch ETL
- **Microservices** — Event streams decouple services
- **Messaging** — Streaming platforms complement (not replace) message brokers

---

## Resources

### Essential Reading
- *Kafka: The Definitive Guide* (2nd ed.) — Shapira, Palino, et al.
- *Designing Event-Driven Systems* — Ben Stopford (free)
- *Streaming Systems* — Akidau, Chernyak, Lax

### Online Resources
- Confluent Developer documentation
- Apache Flink documentation
- Debezium (Change Data Capture) documentation

### Practice Exercises
1. Set up Kafka with 3 partitions and produce/consume events
2. Implement CDC from PostgreSQL to Kafka using Debezium
3. Build a real-time aggregation with Kafka Streams or ksqlDB
4. Configure Schema Registry and evolve an event schema
5. Monitor consumer lag and set up alerting
