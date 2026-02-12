# Architecture Core - Kilat Pet Delivery
*Architecture patterns, conventions, and design decisions*

## Project Overview
- **Project Name**: Kilat Pet Delivery ("Grab for Pets")
- **Description**: Pet transport platform — owners book deliveries, runners transport pets, shop owners manage pet shops
- **Architecture Style**: Microservices
- **Primary Language**: Go 1.24
- **Framework**: Gin (HTTP), GORM (ORM)
- **API Style**: REST + WebSocket (tracking)
- **Organization**: github.com/Kilat-Pet-Delivery

## Architecture Patterns

### Active Patterns
- [x] **Domain-Driven Design (DDD)** — Bounded contexts per service
- [x] **Clean Architecture** — Layers: handler → application → domain → repository
- [x] **Event-Driven Architecture** — Kafka + CloudEvents for async communication
- [x] **Saga Pattern** — Payment escrow with compensation (release/refund)
- [x] **API Gateway Pattern** — Single entry point on port 8080
- [x] **Database per Service** — Each service owns its PostgreSQL database
- [x] **CQRS (light)** — Separate read/write paths in booking service

### Pattern Details

#### Domain-Driven Design
- **What**: Each service = one bounded context with clear boundaries
- **Where**: All 6 business services
- **How**: domain/model (entities), domain/repository (interfaces), application (use cases), infrastructure (implementations)

#### Saga Pattern (Orchestrated)
- **What**: Distributed transaction for payment escrow lifecycle
- **Where**: service-payment ↔ service-booking
- **How**: Booking confirmed → Payment holds escrow → Delivery confirmed → Payment releases to runner. On cancel → Payment refunds. Each step has compensating action.

#### Event-Driven Architecture
- **What**: Services communicate asynchronously via Kafka events
- **Where**: All services publish/consume events
- **How**: CloudEvents envelope wrapping domain events, published to topic-per-domain

## Communication Patterns

### Synchronous Communication
- **Protocol**: HTTP REST (JSON)
- **Service Discovery**: Docker Compose service names (internal DNS)
- **Timeout Policy**: Default Gin timeouts
- **Client**: Flutter apps → API Gateway → individual services

### Asynchronous Communication
- **Message Broker**: Apache Kafka 3.x
- **Event Format**: CloudEvents specification
- **Serialization**: JSON
- **Delivery Guarantee**: At-least-once (consumer commits after processing)
- **Dead Letter Queue**: Not yet implemented (TODO)
- **Topics**: booking.events, payment.events, runner.events, tracking.events

### API Gateway
- **Technology**: Custom Go + Gin reverse proxy
- **Port**: 8080
- **Features**: Path-based routing, WebSocket proxying, rate limiting (100 req/min), CORS, security headers, aggregated health check
- **Route Prefix**: /api/v1/

## Project Structure

### Standard Service Layout
```
service-[name]/
├── cmd/
│   └── server/
│       └── main.go             # Entry point, DI wiring
├── internal/
│   ├── domain/
│   │   ├── model/              # Entities, value objects
│   │   ├── repository/         # Repository interfaces
│   │   └── service/            # Domain service interfaces
│   ├── application/            # Use cases (business logic)
│   │   └── [name]_service.go
│   ├── repository/             # GORM implementations
│   │   └── [name]_repository.go
│   ├── handler/                # Gin HTTP handlers
│   │   └── [name]_handler.go
│   └── events/                 # Kafka producer/consumer
│       └── kafka_consumer.go
├── Dockerfile
├── go.mod
└── go.sum
```

## Shared Libraries

### lib-common
- **Purpose**: Shared middleware, utilities, and infrastructure code
- **Used by**: All 7 services
- **Key modules**:
  - `middleware/`: Auth, CORS, Logger, RateLimit, Recovery, RequestID, SecurityHeaders
  - `kafka/`: Producer, Consumer, CloudEvents envelope
  - `database/`: GORM connection helper
  - `auth/`: JWT manager, UserRole constants
  - `resilience/`: Circuit breaker patterns
  - `health/`: Health check handler
  - `response/`: Standard JSON response helpers
  - `logger/`: Structured logging

### lib-proto
- **Purpose**: Shared event schemas and DTOs
- **Used by**: All services that publish/consume events
- **Key modules**:
  - `events/booking_events.go`: 7 booking event types + payloads
  - `events/payment_events.go`: 5 payment event types + payloads
  - `events/runner_events.go`: 4 runner event types + payloads
  - `events/tracking_events.go`: 3 tracking event types + payloads
  - `dto/`: Shared DTOs (AddressDTO, etc.)

## Coding Conventions

### Naming Conventions
- **Services**: `service-[name]` (kebab-case)
- **Databases**: `kilat_[service]` (snake_case)
- **API Routes**: `/api/v1/[resource]` (lowercase, plural)
- **Events**: `[domain].[action]` (e.g., `booking.requested`, `payment.escrow_held`)
- **Tables**: Auto-created by GORM (plural snake_case)
- **Booking Numbers**: `BK-XXXXXX` format

### Error Handling
- **Pattern**: Per-handler with standard response helper
- **Format**:
```json
{
  "success": false,
  "error": "Human readable message"
}
```

### Authentication & Authorization
- **Method**: JWT (HS256)
- **Token Location**: Header: Authorization Bearer
- **Roles**: owner, runner, admin, shop
- **Auth Service**: service-identity (port 8004)
- **Access Token TTL**: 15 minutes
- **Refresh Token TTL**: 7 days

### Logging
- **Library**: Custom structured logger (lib-common/logger)
- **Format**: Structured with request ID correlation
- **Middleware**: LoggerMiddleware logs all requests

## Architecture Decision Records

### ADR-001: Database per Service
- **Status**: Accepted
- **Decision**: Each service gets its own PostgreSQL database
- **Consequence**: Data isolation, independent scaling, but no cross-service joins

### ADR-002: Kafka for Async Communication
- **Status**: Accepted
- **Decision**: Use Apache Kafka with CloudEvents for inter-service events
- **Consequence**: Loose coupling, event replay capability, but added infrastructure complexity

### ADR-003: Escrow Payment with Saga
- **Status**: Accepted
- **Decision**: Payment uses escrow pattern with Saga compensation
- **Consequence**: Reliable distributed transactions, but complex flow to debug

### ADR-004: PostGIS for Geospatial
- **Status**: Accepted
- **Decision**: Use PostGIS extension for runner/shop nearby search
- **Consequence**: Efficient geospatial queries, but requires PostGIS-enabled PostgreSQL

### ADR-005: WebSocket for Live Tracking
- **Status**: Accepted
- **Decision**: Use WebSocket hub for real-time GPS tracking (not polling)
- **Consequence**: True real-time updates, but JWT passed as query param for WS

## Technical Debt & Known Issues

| Issue | Severity | Affected Services | Notes |
|-------|----------|-------------------|-------|
| Missing owner_id in some events | Medium | notification | EscrowHeld, PaymentFailed, TrackingStarted skip notifications |
| No Dead Letter Queue | Low | All consumers | Failed messages not captured |
| runner.online/offline not consumed | Low | runner, tracking | Events published but no consumer |
| SQL migration cleanup needed | Low | All | GORM auto-migrate, no versioned migrations |
| Integration tests | Medium | All | Not yet implemented |
