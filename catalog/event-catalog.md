# Event Catalog - [PROJECT_NAME]
*Complete reference of all async events across all services*

## Event System Overview

**Message Broker**: [Apache Kafka / RabbitMQ / NATS / Redis Streams]
**Event Format**: [CloudEvents / Custom / Avro / Protobuf]
**Serialization**: [JSON / Protobuf / Avro]
**Total Events**: [TOTAL_COUNT]
**Total Topics**: [TOPIC_COUNT]

## Event Naming Convention

**Pattern**: `[domain].[entity].[action]`

Examples:
- `booking.order.created`
- `payment.transaction.completed`
- `user.account.verified`
- `notification.email.sent`

## Topics / Queues

| # | Topic Name | Partitions | Retention | Consumer Groups |
|---|-----------|-----------|-----------|----------------|
| 1 | [TOPIC_1] | [COUNT] | [DURATION] | [GROUPS] |
| 2 | [TOPIC_2] | [COUNT] | [DURATION] | [GROUPS] |
| 3 | [TOPIC_3] | [COUNT] | [DURATION] | [GROUPS] |
| 4 | [TOPIC_4] | [COUNT] | [DURATION] | [GROUPS] |

---

## Event Registry

### Event: [domain.entity.created]
**Topic**: [TOPIC_NAME]
**Producer**: [SERVICE_NAME]
**Consumer(s)**: [SERVICE_1], [SERVICE_2]
**Trigger**: [When this event is emitted]
**Idempotent**: [Yes/No]

**Payload Schema**:
```json
{
  "event_type": "[domain.entity.created]",
  "event_id": "uuid",
  "timestamp": "ISO-8601",
  "source": "[SERVICE_NAME]",
  "data": {
    "id": "uuid",
    "field_1": "string",
    "field_2": 123,
    "status": "string"
  }
}
```

**Consumer Actions**:
| Consumer | Action on Receive |
|----------|-------------------|
| [SERVICE_1] | [What this service does when it receives this event] |
| [SERVICE_2] | [What this service does when it receives this event] |

---

### Event: [domain.entity.updated]
**Topic**: [TOPIC_NAME]
**Producer**: [SERVICE_NAME]
**Consumer(s)**: [SERVICE_1]
**Trigger**: [When this event is emitted]

**Payload Schema**:
```json
{
  "event_type": "[domain.entity.updated]",
  "event_id": "uuid",
  "timestamp": "ISO-8601",
  "source": "[SERVICE_NAME]",
  "data": {
    "id": "uuid",
    "changes": {}
  }
}
```

---

*Copy the event template above for each additional event.*

## Event Flow Diagrams

### Flow: [Business Process Name]
*Example: Order Processing Flow*

```
[SERVICE_A]                    [SERVICE_B]                    [SERVICE_C]
     |                              |                              |
     | -- [event.created] -------> |                              |
     |                              |                              |
     |                              | -- [event.processed] -----> |
     |                              |                              |
     | <---- [event.completed] --- |                              |
     |                              |                              |
```

### Flow: [Business Process Name 2]
```
[SERVICE_X] --> [event.1] --> [SERVICE_Y] --> [event.2] --> [SERVICE_Z]
                                                    |
                                              [event.3]
                                                    |
                                              [SERVICE_W]
```

## Saga Patterns (If Used)

### Saga: [Saga Name]
**Orchestrator**: [SERVICE_NAME] (or Choreography)
**Purpose**: [What distributed transaction this coordinates]

| Step | Service | Action | Compensating Action |
|------|---------|--------|---------------------|
| 1 | [SERVICE_A] | [Action] | [Rollback action] |
| 2 | [SERVICE_B] | [Action] | [Rollback action] |
| 3 | [SERVICE_C] | [Action] | [Rollback action] |

**Success Flow**:
```
Step 1 OK -> Step 2 OK -> Step 3 OK -> SAGA COMPLETE
```

**Failure Flow** (Step 2 fails):
```
Step 1 OK -> Step 2 FAIL -> Compensate Step 1 -> SAGA ROLLED BACK
```

## Dead Letter Queue (DLQ)

| DLQ Topic | Source Topic | Retry Policy | Alert |
|-----------|-------------|-------------|-------|
| [DLQ_1] | [SOURCE_TOPIC] | [N retries, then DLQ] | [Yes/No] |
| [DLQ_2] | [SOURCE_TOPIC] | [N retries, then DLQ] | [Yes/No] |

## Producer/Consumer Matrix

| Event | Producer | Consumer 1 | Consumer 2 | Consumer 3 |
|-------|----------|-----------|-----------|-----------|
| [event.1] | [SVC_A] | [SVC_B] | [SVC_C] | — |
| [event.2] | [SVC_B] | [SVC_A] | — | — |
| [event.3] | [SVC_C] | [SVC_D] | [SVC_E] | [SVC_F] |

## Consumer Groups

| Group ID | Service | Subscribed Topics | Processing |
|----------|---------|-------------------|------------|
| [GROUP_1] | [SERVICE] | [TOPICS] | [Sequential/Parallel] |
| [GROUP_2] | [SERVICE] | [TOPICS] | [Sequential/Parallel] |

## Adding a New Event

When a new event is created:
1. Add entry to the **Event Registry** above
2. Define the payload schema
3. Add to **Producer/Consumer Matrix**
4. Update **Event Flow Diagrams** if part of a flow
5. Register consumer group if new
6. Update topic list if new topic needed
7. Consider DLQ configuration

---

**Version**: Event Catalog v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific events

*This file is the single source of truth for all async events. Keep it updated when event flows change!*
