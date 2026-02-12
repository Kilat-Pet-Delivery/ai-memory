# New Service Protocol - Microservices MemoryCore
*Step-by-step checklist for creating a new microservice*

## Protocol Activation

**Trigger**: User says "new service" or "add service [name]"

**AI Response**: Walks through the checklist below, adapting to the project's tech stack and conventions.

## Pre-Creation Checklist

### 1. Define the Service
- [ ] **Service Name**: `service-[name]` (follow naming convention)
- [ ] **Responsibility**: Clear bounded context description
- [ ] **Port**: Check port allocation map for available port
- [ ] **Database**: Decide if service needs its own DB
- [ ] **Events**: Will it produce or consume events?
- [ ] **Dependencies**: Which existing services will it interact with?

### 2. Verify No Overlap
- [ ] Check `service-registry.md` — does this responsibility already exist?
- [ ] Check `api-catalog.md` — do similar endpoints already exist?
- [ ] Check `event-catalog.md` — are related events already handled?
- [ ] Confirm this is a separate bounded context, not a feature of existing service

## Service Scaffolding

### 3. Create Project Structure

```
service-[name]/
├── cmd/
│   └── main.[ext]              # Entry point
├── internal/
│   ├── domain/
│   │   ├── model/              # Entities, value objects
│   │   │   └── [entity].go
│   │   ├── repository/         # Repository interfaces
│   │   │   └── [entity]_repository.go
│   │   └── service/            # Domain service interfaces
│   │       └── [entity]_service.go
│   ├── application/            # Use cases
│   │   └── [entity]_usecase.go
│   ├── infrastructure/
│   │   ├── database/           # Repository implementations
│   │   │   └── [entity]_repo_impl.go
│   │   ├── messaging/          # Event pub/sub
│   │   │   ├── producer.go
│   │   │   └── consumer.go
│   │   └── external/           # External API clients
│   └── interfaces/
│       ├── http/               # HTTP handlers
│       │   ├── handler.go
│       │   └── routes.go
│       └── consumer/           # Event consumers
│           └── event_handler.go
├── config/
│   └── config.[ext]            # Configuration
├── migrations/
│   └── V001__initial.sql       # First migration
├── Dockerfile                  # Container definition
├── go.mod / package.json       # Dependencies
└── README.md                   # Service documentation
```

*Adjust structure to match your project's conventions from `architecture-core.md`.*

### 4. Core Implementation

#### Entry Point
```
- [ ] Create main entry point
- [ ] Initialize configuration (env vars, config files)
- [ ] Initialize database connection
- [ ] Initialize message broker connection (if needed)
- [ ] Initialize HTTP server
- [ ] Register routes
- [ ] Start event consumers (if needed)
- [ ] Add graceful shutdown handler
- [ ] Add health check endpoint: GET /health
```

#### Domain Layer
```
- [ ] Define domain models/entities
- [ ] Define repository interfaces
- [ ] Define service interfaces
- [ ] Define domain events (if any)
- [ ] Define value objects
```

#### Application Layer
```
- [ ] Implement use cases
- [ ] Add input validation
- [ ] Add business logic
- [ ] Add event publishing (if needed)
```

#### Infrastructure Layer
```
- [ ] Implement repository (database access)
- [ ] Implement event producer (if needed)
- [ ] Implement event consumer (if needed)
- [ ] Implement external service clients (if needed)
```

#### Interface Layer
```
- [ ] Implement HTTP handlers
- [ ] Define routes
- [ ] Add middleware (auth, logging, cors)
- [ ] Add request/response DTOs
- [ ] Implement event handlers (if consuming events)
```

### 5. Database Setup

```
- [ ] Choose database name: [project]_[service]
- [ ] Add to init-databases.sql (or equivalent)
- [ ] Create initial migration
- [ ] Add required extensions (uuid-ossp, PostGIS, etc.)
- [ ] Define tables and indexes
- [ ] Add seed data if needed
```

### 6. Containerization

#### Dockerfile
```dockerfile
# Multi-stage build example
FROM [BASE_IMAGE] AS builder
WORKDIR /app
COPY . .
RUN [BUILD_COMMAND]

FROM [RUNTIME_IMAGE]
COPY --from=builder /app/[binary] /app/
EXPOSE [PORT]
CMD ["/app/[binary]"]
```

#### Docker Compose Entry
```yaml
service-[name]:
  build:
    context: [BUILD_CONTEXT]
    dockerfile: service-[name]/Dockerfile
  ports:
    - "[PORT]:[PORT]"
  environment:
    - DB_HOST=[DB_HOST]
    - DB_PORT=[DB_PORT]
    - DB_NAME=[DB_NAME]
    - BROKER_HOST=[BROKER_HOST]
  depends_on:
    - [database]
    - [broker]
  networks:
    - [NETWORK_NAME]
```

### 7. API Gateway Integration

```
- [ ] Add route to API gateway configuration
- [ ] Define path prefix: /api/v1/[resource]
- [ ] Configure auth requirement
- [ ] Configure rate limiting (if needed)
- [ ] Test routing from gateway to service
```

## Post-Creation Checklist

### 8. Update Memory Files

**CRITICAL**: Update all MemoryCore files to reflect the new service.

```
- [ ] service-registry.md — Add service to registry table + detailed profile
- [ ] master-memory.md — Update quick reference table + service count
- [ ] api-catalog.md — Add all endpoints
- [ ] event-catalog.md — Add produced/consumed events
- [ ] database-catalog.md — Add database and table schemas
- [ ] infrastructure.md — Add to Docker Compose section + port map
- [ ] architecture-core.md — Update if new patterns introduced
```

### 9. Integration Verification

```
- [ ] Service starts without errors
- [ ] Health check returns 200
- [ ] Database connection works
- [ ] API endpoints respond correctly
- [ ] Events publish successfully (if applicable)
- [ ] Events consume successfully (if applicable)
- [ ] Gateway routes to service correctly
- [ ] Auth middleware works (if applicable)
- [ ] Service appears in docker-compose ps
```

### 10. Documentation

```
- [ ] Service README.md written
- [ ] API endpoints documented
- [ ] Environment variables documented
- [ ] Dependencies listed
- [ ] Run instructions included
```

## Quick Reference: New Service Template

```
Service: service-[name]
Port: [PORT]
Database: [project]_[name]
Gateway Route: /api/v1/[resource]
Health: GET /health
Events Produced: [list]
Events Consumed: [list]
Depends On: [services]
```

---

**Protocol Status**: Ready
**Activation**: "new service" or "add service [name]"
**Output**: Fully scaffolded, documented, and registered microservice

*Follow this protocol to ensure every new service is properly created, configured, and documented!*
