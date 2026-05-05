---
name: streaming
description: Kafka topic conventions, consumer and producer patterns, and event schema standards.
---
# streaming.md

## Purpose

Define conventions for real-time data streaming — covering Kafka topic design,
consumer/producer patterns, event schema conventions, exactly-once semantics,
and modular architecture for event-driven pipelines.

---

## Conventions

### When to Use Streaming vs Batch

| Criteria | Batch (ETL) | Streaming |
|---|---|---|
| Latency requirement | Minutes to hours | Seconds to sub-second |
| Data volume pattern | Periodic bulk | Continuous flow |
| Source type | Database, API, file | Event, webhook, log |
| Processing complexity | Complex aggregations | Simple transforms, routing |
| Cost sensitivity | Lower (scheduled) | Higher (always-on) |

### Topic Architecture

```
/events
  /schemas
    order_created.avsc     # Avro schema definitions
    order_updated.avsc
    payment_received.avsc
  /producers
    order_producer.py      # Domain-specific producers
    payment_producer.py
  /consumers
    order_consumer.py      # Domain-specific consumers
    analytics_consumer.py
  /config
    kafka_config.yaml      # Broker, topic, consumer group config
```

### Topic Naming Conventions

Format: `{domain}.{entity}.{event_type}`

| Element | Convention | Example |
|---|---|---|
| Domain | Business domain | `orders`, `payments`, `users` |
| Entity | The thing the event is about | `order`, `payment`, `user` |
| Event type | What happened | `created`, `updated`, `deleted` |
| Full topic | Dot-separated | `orders.order.created` |
| DLQ topic | Append `.dlq` | `orders.order.created.dlq` |
| Retry topic | Append `.retry` | `orders.order.created.retry` |

### Event Schema Conventions

Every event must include a standard envelope:

```json
{
  "event_id": "uuid-v4",
  "event_type": "order.created",
  "event_version": "1.0",
  "source": "shopify-integration",
  "timestamp": "2026-01-15T10:30:00Z",
  "correlation_id": "uuid-for-tracing",
  "data": {
    "order_id": "12345",
    "customer_email": "user@example.com",
    "total_amount": 99.99,
    "currency": "EUR"
  }
}
```

**Schema rules:**
- Always include `event_id` (UUID v4) for deduplication
- Always include `event_version` for schema evolution
- Always include `timestamp` in ISO 8601 UTC
- Always include `correlation_id` for distributed tracing
- Payload goes in `data` — never mix metadata with business data

### Producer Conventions

```python
# producers/order_producer.py
from confluent_kafka import Producer
from shared.schemas import OrderCreatedEvent
import json, uuid

class OrderProducer:
    def __init__(self, config: dict):
        self.producer = Producer(config)
        self.topic = "orders.order.created"

    def publish(self, order_data: dict) -> str:
        event = OrderCreatedEvent(
            event_id=str(uuid.uuid4()),
            event_type="order.created",
            event_version="1.0",
            source="order-service",
            data=order_data,
        )
        self.producer.produce(
            topic=self.topic,
            key=order_data["order_id"],      # Partition by order ID
            value=json.dumps(event.dict()),
            callback=self._delivery_callback,
        )
        self.producer.flush()
        return event.event_id

    def _delivery_callback(self, err, msg):
        if err:
            logger.error(f"Delivery failed: {err}")
        else:
            logger.info(f"Delivered to {msg.topic()} [{msg.partition()}]")
```

### Consumer Conventions

```python
# consumers/order_consumer.py
from confluent_kafka import Consumer, KafkaError

class OrderConsumer:
    def __init__(self, config: dict):
        self.consumer = Consumer({
            **config,
            "group.id": "order-processing-group",
            "auto.offset.reset": "earliest",
            "enable.auto.commit": False,       # Manual commit for at-least-once
        })
        self.consumer.subscribe(["orders.order.created"])

    def process_messages(self):
        while True:
            msg = self.consumer.poll(timeout=1.0)
            if msg is None:
                continue
            if msg.error():
                self._handle_error(msg)
                continue

            try:
                event = json.loads(msg.value())
                if self._is_duplicate(event["event_id"]):
                    logger.info(f"Skipping duplicate: {event['event_id']}")
                    self.consumer.commit(msg)
                    continue

                self._process_event(event)
                self._mark_processed(event["event_id"])
                self.consumer.commit(msg)       # Commit after successful processing
            except Exception as e:
                self._send_to_dlq(msg, error=str(e))
                self.consumer.commit(msg)
```

### Partitioning Strategy

| Use Case | Partition Key | Rationale |
|---|---|---|
| Order events | `order_id` | All events for one order go to same partition |
| User events | `user_id` | Ordering per user preserved |
| Multi-tenant | `tenant_id` | Isolation between tenants |
| Generic logs | None (round-robin) | Even distribution, no ordering needed |

### Delivery Guarantees

| Guarantee | How | Trade-off |
|---|---|---|
| At-most-once | Auto-commit before processing | Fast but may lose messages |
| At-least-once | Manual commit after processing | Safe but may duplicate |
| Exactly-once | Idempotent producer + transactional consumer | Safest but slowest |

**Default: at-least-once** with idempotent consumers (deduplication via `event_id`).

### Consumer Group Conventions

- One consumer group per distinct processing concern
- Name format: `{service}-{purpose}-group` → `analytics-aggregation-group`
- Scale consumers horizontally by adding instances to the same group
- Monitor consumer lag — alert if lag exceeds threshold

---

## Anti-Patterns

- Never mix metadata with business data in event payloads — use the envelope pattern
- Never use auto-commit in production consumers — manual commit after processing
- Never produce events without a schema version — schema evolution will break consumers
- Never ignore consumer lag — it indicates processing falling behind
- Never use a single topic for all events — separate by domain and entity
- Never skip DLQ routing for failed messages — unprocessable messages must be recoverable
- Never use random partition keys — use domain keys for ordering guarantees
- Never deploy schema changes without backward compatibility

---

## Cross-References

- **data-engineering/etl-pipeline** → streaming as an alternative extract mechanism
- **data-engineering/data-quality** → validate events at consumer ingestion
- **core/observability** → consumer lag monitoring and alerting

---

## Ready-to-Use Prompt

```
Task: Implement streaming pipeline for [event domain]
Skill: data-engineering/streaming, data-engineering/data-quality

REQUIREMENTS:
- Events: [list event types]
- Source: [webhook / service / log]
- Consumers: [list processing concerns]
- Delivery guarantee: [at-least-once / exactly-once]

CONSTRAINTS:
- Standard event envelope with event_id, version, timestamp, correlation_id
- Partition by domain key (not random)
- Manual commit after processing (no auto-commit)
- Idempotent consumers with event_id deduplication
- DLQ for unprocessable messages
- Consumer group per processing concern
- Schema versioning for evolution

DONE WHEN:
- Producer publishes events with standard envelope
- Consumer processes with correct delivery guarantee
- DLQ configured for failed messages
- Consumer lag monitoring in place
- Schema documented and versioned
- Code review skill applied before commit
```

