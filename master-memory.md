# Microservices MemoryCore - Kilat Pet Delivery
*Entry point for instant architecture restoration*

## System Declaration
**I am your Kilat Pet Delivery Architecture AI** — I maintain deep, persistent knowledge of the **Kilat Pet Delivery** microservices system ("Grab for Pets"). I remember every service, endpoint, event, database, and infrastructure component across all conversations.

## Core Loading System

### Instant Restoration Protocol
When you type **"load microservices"** in any conversation:

1. **Load architecture patterns** from `main/architecture-core.md`
2. **Load service registry** from `main/service-registry.md`
3. **Load infrastructure** from `main/infrastructure.md`
4. **Restore session context** from `main/current-session.md`
5. **READY** — Full Kilat architecture knowledge restored!

### Quick Commands
```
"load microservices"    -> Full system restoration
"load services"         -> Service registry only
"load architecture"     -> Architecture patterns only
"load infrastructure"   -> Infrastructure config only
"load api catalog"      -> All 40+ API endpoints
"load events"           -> 19 Kafka events across 4 topics
"load databases"        -> 6 databases, 10 tables
"debug [service]"       -> Debug context for specific service
"load ddd"              -> Bounded contexts, aggregates, domain events
"load security"         -> JWT, RBAC, OWASP protection
"save"                  -> Persist all changes to files
```

## Essential Components (Always Load)

### [Architecture Core](./main/architecture-core.md)
- Go 1.24 + Gin + GORM, DDD, Clean Architecture, Event-Driven
- Kafka + CloudEvents, Saga Pattern, API Gateway
- Shared libraries: lib-common, lib-proto
- **ESSENTIAL** — This defines HOW the system is built

### [Service Registry](./main/service-registry.md)
- 7 services: identity, booking, payment, runner, tracking, notification, gateway
- Ports 8001-8006 + gateway 8080
- PostgreSQL per service, PostGIS for geospatial
- **ESSENTIAL** — This defines WHAT services exist

### [Infrastructure](./main/infrastructure.md)
- Docker Compose at infrastructure/docker-compose.yml
- PostgreSQL 16 + PostGIS 3.4 (port 5433), Kafka (9092), Zookeeper (2181)
- API Gateway on port 8080
- **ESSENTIAL** — This defines WHERE services run

### [Current Session](./main/current-session.md)
- Working memory for active development
- Current service being worked on
- Recent changes and debugging context
- **ESSENTIAL** — This is session RAM

## Quick Reference — Service Registry

| Service | Port | Database | Status |
|---------|------|----------|--------|
| service-identity | 8004 | kilat_identity | Active |
| service-booking | 8001 | kilat_booking (PostGIS) | Active |
| service-payment | 8002 | kilat_payment | Active |
| service-runner | 8003 | kilat_runner (PostGIS) | Active |
| service-tracking | 8005 | kilat_tracking (PostGIS) | Active |
| service-notification | 8006 | kilat_notification | Active |
| api-gateway | 8080 | — (proxy only) | Active |

## On-Demand Components (Load When Needed)

### API Catalog
*Load when you say: "load api catalog"*
- [API Catalog](./catalog/api-catalog.md) — 40+ endpoints across 6 services + WebSocket

### Event Catalog
*Load when you say: "load events"*
- [Event Catalog](./catalog/event-catalog.md) — 19 Kafka events, 4 topics, Saga pattern

### Database Catalog
*Load when you say: "load databases"*
- [Database Catalog](./catalog/database-catalog.md) — 6 databases, 10 tables, PostGIS + JSONB

### Protocols
- [Debug Protocol](./protocols/debug-protocol.md) — Distributed debugging guide
- [New Service Protocol](./protocols/new-service-protocol.md) — Service scaffolding checklist
- [Incident Protocol](./protocols/incident-protocol.md) — Incident response procedures
- [DDD Protocol](./protocols/ddd-protocol.md) — 6 bounded contexts, aggregates, domain events
- [Security Protocol](./protocols/security-protocol.md) — JWT HS256, 4 roles, OWASP Top 10

## System Status
- **Project**: Kilat Pet Delivery ("Grab for Pets")
- **Services**: 7 microservices (6 business + 1 gateway)
- **Events**: 19 Kafka events across 4 topics
- **Databases**: 6 PostgreSQL databases, 10 tables
- **Apps**: 3 Flutter apps (Owner, Runner, Shop) + 3 Next.js web apps
- **Status**: Code complete, all APKs built, backend healthy

---

**Type "load microservices" to instantly restore full Kilat architecture knowledge!**
