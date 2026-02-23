# Messaging and Event Streaming

## Profile

### What It Is
Messaging and Event Streaming are communication paradigms for distributed systems. Message brokers (RabbitMQ, ActiveMQ) handle point-to-point and publish-subscribe messaging. Event streaming platforms (Kafka, Pulsar, Redpanda) provide durable, ordered, replayable event logs. Both decouple producers from consumers and enable asynchronous communication.

### Origin and Evolution
- Message-oriented middleware dates to the 1980s-90s (IBM MQ, TIBCO)
- AMQP standardized messaging protocols (RabbitMQ, 2007)
- Apache Kafka (2011, LinkedIn) revolutionized event streaming with log-based architecture
- Modern alternatives: Apache Pulsar, Redpanda, NATS, AWS Kinesis
- Convergence trend: platforms offering both messaging and streaming capabilities

### Key Authors and References
- **Gregor Hohpe & Bobby Woolf** — *Enterprise Integration Patterns* (2003)
- **Jay Kreps** — "The Log" blog post, Kafka co-creator
- **Ben Stopford** — *Designing Event-Driven Systems* (Confluent)
- **Martin Kleppmann** — Stream processing in *Designing Data-Intensive Applications*

### Key Concepts at a Glance
| Concept | Message Broker | Event Stream |
|---------|---------------|--------------|
| Model | Queue/Topic | Append-only log |
| Delivery | Message consumed, then deleted | Events retained, replayable |
| Ordering | Per-queue (limited) | Per-partition (strict) |
| Consumer | Competing consumers | Consumer groups with offsets |
| Replay | Not possible | Core feature |
| Example | RabbitMQ, ActiveMQ | Kafka, Pulsar, Redpanda |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Decouple producers from consumers** — Producers don't know who consumes their messages. New consumers can be added without changing producers.
2. **Buffer against load spikes** — Messages queue when consumers can't keep up, smoothing traffic spikes instead of dropping requests.
3. **Guarantee delivery semantics** — Understand and choose between at-most-once, at-least-once, and exactly-once delivery based on requirements.
4. **Ordered logs for event history** — Event streaming platforms maintain an ordered, immutable log enabling replay, reprocessing, and temporal queries.
5. **Right tool for the pattern** — Message brokers for task distribution and routing; event streaming for event sourcing, log aggregation, and stream processing.

### When to Use vs. When to Avoid

**Message Broker (RabbitMQ, NATS):**
- Task distribution (work queues)
- Complex routing (topic exchanges, headers)
- Request-reply patterns
- Small messages, transient workloads

**Event Streaming (Kafka, Pulsar):**
- Event sourcing and event-driven architecture
- Log aggregation and audit trails
- Stream processing and analytics
- High-throughput, durable event storage

---

## Frameworks and Methodologies

### 1. Messaging Pattern Selection — step-by-step
1. Identify the communication need: fire-and-forget, request-reply, pub-sub, streaming
2. Determine delivery guarantee requirements (at-most-once, at-least-once, exactly-once)
3. Assess throughput and latency requirements
4. Evaluate ordering requirements (none, per-key, total)
5. Choose technology: broker for transient tasks, streaming for durable events
6. Design consumer idempotency regardless of delivery guarantee

### 2. Kafka Topic Design — step-by-step
1. Define topics by event type or domain (orders, payments, inventory)
2. Choose partition key for ordering (e.g., order_id, customer_id)
3. Set partition count based on consumer parallelism needs
4. Configure retention (time-based, size-based, or compacted)
5. Design consumer groups for independent processing
6. Plan schema evolution with Schema Registry (Avro, Protobuf, JSON Schema)

---

## How to Apply

### Decision Checklist
- [ ] What delivery guarantee does this use case need?
- [ ] Is message ordering required? By what key?
- [ ] Do consumers need to replay historical messages?
- [ ] What throughput (messages/sec) is expected?
- [ ] Is there a schema evolution strategy?
- [ ] Are consumers idempotent?

### Implementation Patterns

**Kafka Producer and Consumer:**
```python
# Producer
from confluent_kafka import Producer

producer = Producer({"bootstrap.servers": "kafka:9092"})

def publish_order_event(event: OrderPlaced) -> None:
    producer.produce(
        topic="orders",
        key=str(event.order_id).encode(),    # Partition by order_id
        value=event.to_json().encode(),
        headers={"event_type": "OrderPlaced", "version": "1"},
    )
    producer.flush()

# Consumer
from confluent_kafka import Consumer

consumer = Consumer({
    "bootstrap.servers": "kafka:9092",
    "group.id": "shipping-service",
    "auto.offset.reset": "earliest",
    "enable.auto.commit": False,
})
consumer.subscribe(["orders"])

while True:
    msg = consumer.poll(timeout=1.0)
    if msg is None:
        continue
    if msg.error():
        handle_error(msg.error())
        continue
    process_event(msg)
    consumer.commit(msg)  # Manual commit after processing
```

**RabbitMQ with Routing:**
```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters("rabbitmq"))
channel = connection.channel()

# Declare exchange and queues
channel.exchange_declare(exchange="notifications", exchange_type="topic")
channel.queue_declare(queue="email-notifications")
channel.queue_declare(queue="sms-notifications")

# Bind with routing keys
channel.queue_bind(queue="email-notifications", exchange="notifications",
                   routing_key="notification.email.*")
channel.queue_bind(queue="sms-notifications", exchange="notifications",
                   routing_key="notification.sms.*")

# Publish with routing
channel.basic_publish(
    exchange="notifications",
    routing_key="notification.email.order_confirmed",
    body=json.dumps({"to": "user@example.com", "template": "order_confirmed"}),
)
```

**Dead Letter Queue Pattern:**
```python
class ResilientConsumer:
    def __init__(self, main_queue: Queue, dlq: Queue, max_retries: int = 3):
        self.main_queue = main_queue
        self.dlq = dlq
        self.max_retries = max_retries

    def process(self, message: Message) -> None:
        retry_count = message.headers.get("x-retry-count", 0)
        try:
            self.handle(message)
            self.main_queue.ack(message)
        except TransientError:
            if retry_count < self.max_retries:
                self.main_queue.nack(message, requeue=True,
                    headers={"x-retry-count": retry_count + 1})
            else:
                self.dlq.publish(message)
                self.main_queue.ack(message)
        except PermanentError:
            self.dlq.publish(message)
            self.main_queue.ack(message)
```

### Common Mistakes
1. **No dead letter queue** — Failed messages are silently lost or retried infinitely
2. **Wrong partition key** — Choosing a key with low cardinality causes hot partitions
3. **Committing before processing** — Auto-commit can acknowledge messages before they're successfully processed
4. **No schema management** — Breaking schema changes cause consumer failures
5. **Treating Kafka as a message queue** — Using Kafka for request-reply or short-lived task distribution where RabbitMQ is simpler
6. **Too many topics or too few partitions** — Over-provisioning topics or under-provisioning partitions limits throughput

### Integration with Other Topics
- **Event-Driven Architecture** — Messaging/streaming is the infrastructure layer for EDA
- **Event Sourcing** — Kafka as an event store for event-sourced systems
- **Microservices** — Async messaging decouples services
- **CQRS** — Events flow from write side to read projections via messaging
- **Saga Pattern** — Messages/events coordinate saga steps
- **Resilience Patterns** — DLQ, retries, circuit breakers complement messaging

---

## Resources

### Essential Reading
- *Enterprise Integration Patterns* — Hohpe & Woolf
- *Designing Event-Driven Systems* — Ben Stopford (free, Confluent)
- *Kafka: The Definitive Guide* (2nd ed.) — Shapira et al.

### Online Resources
- Confluent Developer documentation
- RabbitMQ tutorials (rabbitmq.com/tutorials)
- Jay Kreps — "The Log: What every software engineer should know" blog post

### Practice Exercises
1. Implement a work queue with competing consumers using RabbitMQ
2. Set up Kafka with 3 partitions and verify ordering within a partition
3. Build a dead letter queue handler with retry logic
4. Compare throughput of RabbitMQ vs. Kafka for 10K messages/sec
5. Implement schema evolution with Avro and Schema Registry
