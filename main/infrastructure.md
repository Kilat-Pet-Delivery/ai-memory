# Infrastructure - Kilat Pet Delivery
*Docker Compose, PostgreSQL, Kafka, and API Gateway*

## Infrastructure Overview

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Container Runtime | Docker | Containerization |
| Orchestration | Docker Compose | Dev environment |
| Database | PostgreSQL 16 + PostGIS 3.4 | Primary data store |
| Message Broker | Apache Kafka 3.x | Async event streaming |
| Coordinator | Zookeeper | Kafka coordination |
| API Gateway | Custom Go + Gin | Request routing |

## Docker Compose

**Config File**: `infrastructure/docker-compose.yml`
**Build Context**: `..` (project root, services at top level)

### Service Containers
| Container | Port Mapping | Depends On |
|-----------|-------------|------------|
| service-identity | 8004:8004 | postgres, kafka |
| service-booking | 8001:8001 | postgres, kafka |
| service-payment | 8002:8002 | postgres, kafka |
| service-runner | 8003:8003 | postgres, kafka |
| service-tracking | 8005:8005 | postgres, kafka |
| service-notification | 8006:8006 | postgres, kafka |
| api-gateway | 8080:8080 | all services |

### Infrastructure Containers
| Container | Image | Port Mapping | Purpose |
|-----------|-------|-------------|---------|
| postgres | postgis/postgis:16-3.4 | 5433:5432 | PostgreSQL + PostGIS |
| kafka | confluentinc/cp-kafka:7.x | 9092:9092 | Message broker |
| zookeeper | confluentinc/cp-zookeeper:7.x | 2181:2181 | Kafka coordination |

### Docker Compose Commands
```bash
# Start all (from infrastructure/ directory)
cd infrastructure && docker-compose up -d

# Start specific service
docker-compose up -d service-booking

# View logs
docker-compose logs -f service-booking

# Rebuild after code changes
docker-compose up -d --build service-booking

# Stop all
docker-compose down

# Reset everything (including database volumes)
docker-compose down -v
```

## Databases

### PostgreSQL Configuration
- **Image**: postgis/postgis:16-3.4
- **Host**: localhost (from host) / postgres (from containers)
- **Port**: 5433 (host) → 5432 (container)
- **Init Script**: `infrastructure/infra/init-databases.sql`

### Database per Service

| Service | Database Name | Extensions |
|---------|--------------|------------|
| service-identity | kilat_identity | uuid-ossp |
| service-booking | kilat_booking | PostGIS, uuid-ossp |
| service-payment | kilat_payment | uuid-ossp |
| service-runner | kilat_runner | PostGIS, uuid-ossp |
| service-tracking | kilat_tracking | PostGIS, uuid-ossp |
| service-notification | kilat_notification | uuid-ossp |

### Database Initialization
```sql
-- infrastructure/infra/init-databases.sql
CREATE DATABASE kilat_identity;
CREATE DATABASE kilat_runner;
CREATE DATABASE kilat_booking;
CREATE DATABASE kilat_payment;
CREATE DATABASE kilat_tracking;
CREATE DATABASE kilat_notification;

-- PostGIS extensions
\c kilat_runner
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

\c kilat_booking
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

\c kilat_tracking
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
-- ... (similar for other DBs)
```

### Migrations
- **Tool**: GORM AutoMigrate (in each service's main.go)
- **Note**: No versioned migration files yet (technical debt)

## Message Broker

### Apache Kafka Configuration
- **Host**: localhost (from host) / kafka (from containers)
- **Port**: 9092
- **Library**: kafka-go
- **Balancer**: LeastBytes
- **Batch Timeout**: 10ms

### Topics

| Topic | Purpose |
|-------|---------|
| booking.events | All booking lifecycle events (7 event types) |
| payment.events | All payment/escrow events (5 event types) |
| runner.events | Runner status and location events (4 event types) |
| tracking.events | Trip tracking events (3 event types) |

### Consumer Groups

| Group ID | Service | Topics |
|----------|---------|--------|
| service-booking-payments | service-booking | payment.events |
| service-payment-bookings | service-payment | booking.events |
| service-tracking-bookings | service-tracking | booking.events |
| service-tracking-runners | service-tracking | runner.events |
| service-notification-bookings | service-notification | booking.events |
| service-notification-payments | service-notification | payment.events |
| service-notification-tracking | service-notification | tracking.events |

## API Gateway

### Configuration
- **Technology**: Custom Go + Gin reverse proxy
- **Port**: 8080
- **Rate Limit**: 100 req/min
- **Routing**: Path-prefix based (NoRoute catch-all handler)

### Route Table

| Path Prefix | Target Service | Port | Auth |
|------------|---------------|------|------|
| /api/v1/auth/* | service-identity | 8004 | Mixed |
| /api/v1/bookings/* | service-booking | 8001 | Yes |
| /api/v1/payments/* | service-payment | 8002 | Yes |
| /api/v1/runners/* | service-runner | 8003 | Yes |
| /api/v1/petshops/* | service-runner | 8003 | Yes |
| /api/v1/tracking/* | service-tracking | 8005 | Yes |
| /api/v1/notifications/* | service-notification | 8006 | Yes |
| /ws/tracking/* | service-tracking | 8005 | WS+JWT |

### Gateway Features
- [x] Path-based reverse proxy
- [x] Rate Limiting (100 req/min)
- [x] CORS Configuration
- [x] Security Headers
- [x] Request ID propagation
- [x] WebSocket Proxying (/ws/tracking/)
- [x] Aggregated Health Check (/health)

## Network Configuration

```
[Flutter/Next.js Apps]
        |
   [API Gateway] (port 8080)
        |
   [Docker Internal Network]
   |       |       |       |       |       |
[Identity][Booking][Payment][Runner][Tracking][Notification]
  :8004    :8001    :8002   :8003   :8005     :8006
                       |
                   [Kafka] (port 9092)
                       |
                 [Zookeeper] (port 2181)
                       |
                 [PostgreSQL] (port 5433)
```

### Ports Summary
| Port | Component |
|------|-----------|
| 8080 | API Gateway |
| 8001-8006 | Business services |
| 5433 | PostgreSQL + PostGIS |
| 9092 | Apache Kafka |
| 2181 | Zookeeper |

## Local Development Setup

### Prerequisites
```bash
Docker >= 24.0
Go >= 1.24
Flutter >= 3.38 (for mobile apps)
Node.js >= 20 (for Next.js web apps)
```

### Quick Start
```bash
# 1. Start infrastructure
cd infrastructure && docker-compose up -d

# 2. Wait for healthy (all services auto-migrate DB)
docker-compose ps

# 3. Verify gateway
curl http://localhost:8080/health

# 4. Flutter apps connect to:
#    Android emulator: 10.0.2.2:8080
#    iOS simulator: localhost:8080
```

### Flutter App Config
- **Android**: `10.0.2.2:8080` (emulator maps to host localhost)
- **iOS**: `localhost:8080`
- **AndroidManifest.xml**: INTERNET + LOCATION permissions + cleartext traffic
- **network_security_config.xml**: Allows 10.0.2.2 + localhost
