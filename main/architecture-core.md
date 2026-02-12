# Architecture Core - [PROJECT_NAME]
*Architecture patterns, conventions, and design decisions*

## Project Overview
- **Project Name**: [PROJECT_NAME]
- **Description**: [BRIEF_DESCRIPTION]
- **Architecture Style**: Microservices
- **Primary Language**: [LANGUAGE] (e.g., Go, Java, Node.js, Python)
- **Framework**: [FRAMEWORK] (e.g., Gin, Spring Boot, Express, FastAPI)
- **API Style**: [API_STYLE] (e.g., REST, gRPC, GraphQL)

## Architecture Patterns

### Active Patterns
*Check all patterns used in this project:*

- [ ] **Domain-Driven Design (DDD)** — Bounded contexts per service
- [ ] **Clean Architecture** — Layers: handler -> service -> repository
- [ ] **Hexagonal Architecture** — Ports and adapters
- [ ] **CQRS** — Separate read/write models
- [ ] **Event Sourcing** — Events as source of truth
- [ ] **Saga Pattern** — Distributed transaction coordination
- [ ] **API Gateway Pattern** — Single entry point for clients
- [ ] **Service Mesh** — Sidecar proxy for service-to-service
- [ ] **Circuit Breaker** — Fault tolerance and resilience
- [ ] **Event-Driven Architecture** — Async communication via events
- [ ] **Database per Service** — Each service owns its data
- [ ] **Strangler Fig** — Gradual migration from monolith

### Pattern Details

#### [PATTERN_1]
- **What**: [Brief description]
- **Where**: [Which services use it]
- **How**: [Implementation details]

#### [PATTERN_2]
- **What**: [Brief description]
- **Where**: [Which services use it]
- **How**: [Implementation details]

## Communication Patterns

### Synchronous Communication
- **Protocol**: [HTTP REST / gRPC / GraphQL]
- **Service Discovery**: [Method — DNS, Consul, K8s service, etc.]
- **Load Balancing**: [Method — client-side, server-side, etc.]
- **Timeout Policy**: [Default timeout for inter-service calls]

### Asynchronous Communication
- **Message Broker**: [Kafka / RabbitMQ / NATS / Redis Streams]
- **Event Format**: [CloudEvents / custom / Avro / Protobuf]
- **Serialization**: [JSON / Protobuf / Avro]
- **Delivery Guarantee**: [At-least-once / exactly-once / at-most-once]
- **Dead Letter Queue**: [Yes/No — handling failed messages]

### API Gateway
- **Technology**: [Kong / Nginx / custom / cloud-native]
- **Port**: [GATEWAY_PORT]
- **Features**: [Routing, rate limiting, auth, CORS, etc.]
- **Route Prefix**: [e.g., /api/v1/]

## Project Structure

### Standard Service Layout
```
service-[name]/
├── cmd/                    # Entry point
│   └── main.[ext]
├── internal/               # Private application code
│   ├── domain/             # Domain models and interfaces
│   │   ├── model/          # Entities and value objects
│   │   ├── repository/     # Repository interfaces
│   │   └── service/        # Domain service interfaces
│   ├── application/        # Use cases / application services
│   ├── infrastructure/     # External implementations
│   │   ├── database/       # Repository implementations
│   │   ├── messaging/      # Event publisher/consumer
│   │   └── external/       # Third-party integrations
│   └── interfaces/         # Entry points
│       ├── http/           # HTTP handlers/controllers
│       └── consumer/       # Event consumers
├── config/                 # Configuration files
├── migrations/             # Database migrations
└── tests/                  # Test files
```

*Adjust this structure to match your project's conventions.*

## Shared Libraries

### [LIB_NAME_1]
- **Purpose**: [What it provides]
- **Used by**: [Which services]
- **Key modules**:
  - [module_1]: [description]
  - [module_2]: [description]
  - [module_3]: [description]

### [LIB_NAME_2]
- **Purpose**: [What it provides]
- **Used by**: [Which services]
- **Key modules**:
  - [module_1]: [description]
  - [module_2]: [description]

## Coding Conventions

### Naming Conventions
- **Services**: `service-[name]` (kebab-case)
- **Databases**: `[project]_[service]` (snake_case)
- **API Routes**: `/api/v1/[resource]` (lowercase, plural)
- **Events**: `[domain].[action].[past-tense]` (e.g., `order.payment.completed`)
- **Tables**: `[plural_noun]` (snake_case)

### Error Handling
- **Pattern**: [Centralized / per-service / middleware]
- **Format**: [Standard error response format]
```json
{
  "error": {
    "code": "[ERROR_CODE]",
    "message": "[Human readable message]",
    "details": {}
  }
}
```

### Authentication & Authorization
- **Method**: [JWT / OAuth2 / API Key / mTLS]
- **Token Location**: [Header: Authorization Bearer / Cookie]
- **Roles**: [List of roles]
- **Auth Service**: [Which service handles auth]

### Logging
- **Library**: [Logger library name]
- **Format**: [JSON / structured / plain]
- **Levels**: [debug, info, warn, error, fatal]
- **Correlation**: [Trace ID / Request ID propagation method]

## Architecture Decision Records (ADR)

### ADR-001: [Decision Title]
- **Date**: [Date]
- **Status**: [Accepted / Deprecated / Superseded]
- **Context**: [Why this decision was needed]
- **Decision**: [What was decided]
- **Consequences**: [Positive and negative outcomes]

### ADR-002: [Decision Title]
- **Date**: [Date]
- **Status**: [Accepted / Deprecated / Superseded]
- **Context**: [Why]
- **Decision**: [What]
- **Consequences**: [Outcomes]

## Technical Debt & Known Issues

| Issue | Severity | Affected Services | Notes |
|-------|----------|-------------------|-------|
| [ISSUE_1] | [High/Med/Low] | [Services] | [Details] |
| [ISSUE_2] | [High/Med/Low] | [Services] | [Details] |

---

**Version**: Architecture Core v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific configuration

*This file defines how your microservices system is built. Update it when architecture decisions change.*
