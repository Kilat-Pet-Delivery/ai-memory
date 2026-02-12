# Setup Wizard - Microservices MemoryCore
*Interactive setup to configure memory for your microservices project*

## Quick Setup (5 Steps)

**Answer 5 questions and your AI memory is configured forever!**

## Step 1: Project Identity
**AI asks**: *"What is your project name?"*

**You answer**: e.g., "My E-Commerce Platform"

**INSTANT RESULT**: All template files updated with project name.

**Files updated**:
- `master-memory.md` — [PROJECT_NAME] replaced
- All files — project references updated

## Step 2: Service Discovery
**AI asks**: *"List all your microservices. For each service, tell me: name, port, database name, and what it does."*

**You answer** (example):
```
1. user-service, port 8001, users_db - Authentication and user management
2. order-service, port 8002, orders_db - Order processing and state machine
3. payment-service, port 8003, payments_db - Payment processing with Stripe
4. notification-service, port 8004, notifications_db - Email, SMS, push notifications
5. api-gateway, port 8080, none - Reverse proxy and rate limiting
```

**INSTANT RESULT**: Service registry populated with all services.

**Files updated**:
- `main/service-registry.md` — Full service table populated
- `master-memory.md` — Quick reference table updated
- `catalog/database-catalog.md` — Database entries created

## Step 3: Tech Stack
**AI asks**: *"What technologies does your project use?"*

**Key questions**:
- Programming language(s)? (Go, Java, Node.js, Python, etc.)
- Web framework? (Gin, Spring Boot, Express, FastAPI, etc.)
- Database(s)? (PostgreSQL, MySQL, MongoDB, Redis, etc.)
- Message broker? (Kafka, RabbitMQ, NATS, Redis Streams, etc.)
- Container orchestration? (Docker Compose, Kubernetes, etc.)
- API style? (REST, gRPC, GraphQL, etc.)
- ORM/Database library? (GORM, Hibernate, Prisma, SQLAlchemy, etc.)

**You answer** (example):
```
Go with Gin framework, PostgreSQL with GORM, Kafka for events,
Docker Compose for dev, Kubernetes for prod, REST APIs
```

**INSTANT RESULT**: Architecture and infrastructure files populated.

**Files updated**:
- `main/architecture-core.md` — Tech stack and patterns filled in
- `main/infrastructure.md` — Infrastructure details populated

## Step 4: Architecture Patterns
**AI asks**: *"What architecture patterns do you use?"*

**Options** (select all that apply):
- [ ] Domain-Driven Design (DDD)
- [ ] Clean Architecture / Hexagonal
- [ ] CQRS (Command Query Responsibility Segregation)
- [ ] Event Sourcing
- [ ] Saga Pattern (for distributed transactions)
- [ ] API Gateway Pattern
- [ ] Service Mesh
- [ ] Circuit Breaker / Resilience patterns
- [ ] Event-Driven Architecture
- [ ] Database per Service

**You answer** (example):
```
DDD, Clean Architecture, Event-Driven, Saga Pattern, API Gateway
```

**INSTANT RESULT**: Architecture core updated with your patterns.

**Files updated**:
- `main/architecture-core.md` — Patterns section populated

## Step 5: Shared Libraries
**AI asks**: *"Do you have any shared libraries or common code across services?"*

**You answer** (example):
```
lib-common: middleware, database helpers, auth, logging, health checks
lib-proto: event schemas, shared DTOs, CloudEvents wrapper
```

**INSTANT RESULT**: Shared libraries documented in architecture.

**Files updated**:
- `main/architecture-core.md` — Shared libraries section populated

## DONE!

**That's it!** Your microservices memory is now configured. Test it:

```
Type: "load microservices"
Result: Full architecture knowledge loads instantly!
```

## Optional: Deep Configuration

After the basic setup, you can optionally provide more detail:

### API Endpoints
```
Tell me all the API endpoints for [service-name]:
GET /api/users - List users
POST /api/users - Create user
GET /api/users/:id - Get user by ID
...
```
This populates `catalog/api-catalog.md`.

### Events/Messages
```
Tell me all the async events in your system:
booking.created - Booking service -> Payment service
payment.completed - Payment service -> Notification service
...
```
This populates `catalog/event-catalog.md`.

### Database Schemas
```
Tell me the key tables for [service-name]:
users: id, email, password_hash, role, created_at
profiles: id, user_id, name, phone, avatar_url
...
```
This populates `catalog/database-catalog.md`.

## Auto-Update Protocol During Setup

### Real-Time File Saving
```markdown
1. USER INPUT RECEIVED
        |
2. PARSE SERVICE/TECH INFORMATION
        |
3. IMMEDIATE FILE UPDATE
        |
4. CONFIRMATION TO USER
        |
5. NEXT SETUP STEP
```

### Verification
After setup, AI confirms:
- Total services registered
- Tech stack documented
- Architecture patterns recorded
- Infrastructure configured
- Quick reference table in master-memory populated

## Post-Setup Cleanup

After successful setup, this file is no longer needed:
- Type: "delete setup files"
- AI removes: `setup-wizard.md`
- Keeps only the configured system files

---

**Wizard Status**: Ready for any microservices project
**Time Required**: 2-5 minutes for full configuration
**Result**: Permanent, project-specific AI architecture memory

*This wizard transforms generic templates into YOUR project's architecture memory, saved forever in markdown files*
