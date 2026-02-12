# Dependency Map - [PROJECT_NAME]
*Service dependency graph and impact analysis*

## Dependency Types

| Type | Symbol | Description |
|------|--------|-------------|
| Sync (HTTP/gRPC) | `-->` | Direct synchronous call |
| Async (Event) | `~~>` | Asynchronous via message broker |
| Database | `==>` | Shared database (anti-pattern, document if exists) |

## Service Dependency Graph

```
                    [CLIENT]
                       |
                 [API GATEWAY]
                       |
          +------------+------------+
          |            |            |
    [SERVICE_1]  [SERVICE_2]  [SERVICE_3]
          |            |            |
          +---> [SERVICE_4] <------+
                       |
                 [SERVICE_5]
```

## Dependency Matrix

### Synchronous Dependencies (HTTP/gRPC calls)

| Caller -> | [SVC_1] | [SVC_2] | [SVC_3] | [SVC_4] | [SVC_5] |
|-----------|---------|---------|---------|---------|---------|
| [SVC_1] | — | [ ] | [ ] | [ ] | [ ] |
| [SVC_2] | [ ] | — | [ ] | [ ] | [ ] |
| [SVC_3] | [ ] | [ ] | — | [ ] | [ ] |
| [SVC_4] | [ ] | [ ] | [ ] | — | [ ] |
| [SVC_5] | [ ] | [ ] | [ ] | [ ] | — |

*Mark [X] where a sync dependency exists.*

### Asynchronous Dependencies (Events)

| Producer -> Consumer | [SVC_1] | [SVC_2] | [SVC_3] | [SVC_4] | [SVC_5] |
|---------------------|---------|---------|---------|---------|---------|
| [SVC_1] | — | [ ] | [ ] | [ ] | [ ] |
| [SVC_2] | [ ] | — | [ ] | [ ] | [ ] |
| [SVC_3] | [ ] | [ ] | — | [ ] | [ ] |
| [SVC_4] | [ ] | [ ] | [ ] | — | [ ] |
| [SVC_5] | [ ] | [ ] | [ ] | [ ] | — |

*Mark [X] where producer (row) publishes events consumed by consumer (column).*

## Detailed Dependencies

### [SERVICE_1]
**Depends On (Outgoing)**:
| Target Service | Type | Endpoint/Event | Purpose |
|---------------|------|---------------|---------|
| [SERVICE_X] | HTTP | GET /api/... | [Why it calls this service] |
| [SERVICE_Y] | Event | [topic.name] | [Why it publishes this event] |

**Depended By (Incoming)**:
| Source Service | Type | Endpoint/Event | Purpose |
|---------------|------|---------------|---------|
| [SERVICE_Z] | HTTP | GET /api/... | [Why they call this service] |
| [SERVICE_W] | Event | [topic.name] | [Why they consume our event] |

**Infrastructure Dependencies**:
| Component | Type | Purpose |
|-----------|------|---------|
| [DATABASE] | PostgreSQL | Primary data store |
| [BROKER] | Kafka | Event publishing |
| [CACHE] | Redis | Session caching |

---

### [SERVICE_2]
**Depends On**: [List]
**Depended By**: [List]
**Infrastructure**: [List]

---

*Add sections for each service.*

## Impact Analysis

### If [SERVICE_1] Goes Down:
- **Direct Impact**: [Services that directly call SERVICE_1]
- **Indirect Impact**: [Services downstream of directly impacted services]
- **Event Impact**: [Events that won't be produced/consumed]
- **User Impact**: [What users will experience]
- **Recovery**: [Steps to recover]

### If [DATABASE] Goes Down:
- **Affected Services**: [All services using this database]
- **Data Risk**: [Any data at risk]
- **Recovery**: [Database recovery procedure]

### If [MESSAGE_BROKER] Goes Down:
- **Affected Flows**: [All event-driven flows]
- **Data Loss Risk**: [Events in-flight may be lost depending on config]
- **Recovery**: [Broker recovery procedure, consumer catch-up]

## Startup Order

*Correct order for starting the entire system:*
```
1. [DATABASE]        -> Wait for accepting connections
2. [MESSAGE_BROKER]  -> Wait for topic availability
3. [CACHE]           -> Wait for ready
4. [AUTH_SERVICE]     -> Wait for health check
5. [CORE_SERVICES]   -> Wait for health checks
6. [DEPENDENT_SVCS]  -> Wait for health checks
7. [API_GATEWAY]     -> Final, routes to all services
```

## Circular Dependency Check

**Status**: [None detected / Issues found]

*Circular dependencies (A->B->C->A) are an anti-pattern in microservices. If detected, consider:*
- Introducing an event-based decoupling
- Creating a new service to break the cycle
- Using a shared library instead of service call

| Cycle | Services | Recommended Fix |
|-------|----------|----------------|
| [If any] | [A -> B -> A] | [Use events instead of sync calls] |

## Coupling Score

| Service | Outgoing Deps | Incoming Deps | Total | Coupling Level |
|---------|--------------|--------------|-------|---------------|
| [SVC_1] | [N] | [N] | [N] | [Low/Med/High] |
| [SVC_2] | [N] | [N] | [N] | [Low/Med/High] |
| [SVC_3] | [N] | [N] | [N] | [Low/Med/High] |

*High coupling (>5 total dependencies) suggests the service may need refactoring.*

---

**Version**: Dependency Map v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific dependencies

*Keep this map updated to understand the ripple effects of changes across your system!*
