# Infrastructure - [PROJECT_NAME]
*Container orchestration, databases, message brokers, CI/CD, and monitoring*

## Infrastructure Overview

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Container Runtime | [Docker / Podman] | Containerization |
| Orchestration | [Docker Compose / Kubernetes / ECS] | Container management |
| Database | [PostgreSQL / MySQL / MongoDB] | Primary data store |
| Cache | [Redis / Memcached / None] | Caching layer |
| Message Broker | [Kafka / RabbitMQ / NATS] | Async messaging |
| API Gateway | [Kong / Nginx / Custom / Cloud] | Request routing |
| CI/CD | [GitHub Actions / GitLab CI / Jenkins] | Build & deploy |
| Monitoring | [Prometheus / Datadog / CloudWatch] | Observability |
| Logging | [ELK / Loki / CloudWatch Logs] | Log aggregation |

## Container Orchestration

### Development Environment
**Technology**: [Docker Compose / Minikube / Kind]
**Config File**: [Path to docker-compose.yml or similar]

#### Service Containers
| Container | Image | Port Mapping | Depends On |
|-----------|-------|-------------|------------|
| [SERVICE_1] | [IMAGE] | [HOST:CONTAINER] | [DB, BROKER] |
| [SERVICE_2] | [IMAGE] | [HOST:CONTAINER] | [DB, BROKER] |
| [SERVICE_3] | [IMAGE] | [HOST:CONTAINER] | [DB, BROKER] |

#### Infrastructure Containers
| Container | Image | Port Mapping | Purpose |
|-----------|-------|-------------|---------|
| [DATABASE] | [IMAGE:TAG] | [HOST:CONTAINER] | Primary database |
| [BROKER] | [IMAGE:TAG] | [HOST:CONTAINER] | Message broker |
| [CACHE] | [IMAGE:TAG] | [HOST:CONTAINER] | Cache layer |
| [GATEWAY] | [IMAGE:TAG] | [HOST:CONTAINER] | API gateway |

#### Docker Compose Commands
```bash
# Start all services
docker-compose up -d

# Start specific service
docker-compose up -d [service-name]

# View logs
docker-compose logs -f [service-name]

# Rebuild after code changes
docker-compose up -d --build [service-name]

# Stop all services
docker-compose down

# Reset everything (including volumes)
docker-compose down -v
```

### Production Environment
**Technology**: [Kubernetes / ECS / Cloud Run]
**Config Location**: [Path to k8s manifests or similar]

#### Cluster Configuration
- **Nodes**: [Count and type]
- **Namespaces**: [List of namespaces]
- **Ingress**: [Ingress controller type]
- **TLS**: [Certificate management method]

## Databases

### Primary Database
- **Technology**: [PostgreSQL 16 / MySQL 8 / MongoDB 7]
- **Host**: [DB_HOST]
- **Port**: [DB_PORT]
- **Connection**: [Connection string pattern]
- **Extensions**: [PostGIS, uuid-ossp, etc.]

### Database per Service

| Service | Database Name | Extensions | Notes |
|---------|--------------|------------|-------|
| [SERVICE_1] | [DB_NAME_1] | [Extensions] | [Notes] |
| [SERVICE_2] | [DB_NAME_2] | [Extensions] | [Notes] |
| [SERVICE_3] | [DB_NAME_3] | [Extensions] | [Notes] |

### Database Initialization
```sql
-- Example: init-databases.sql
CREATE DATABASE [DB_NAME_1];
CREATE DATABASE [DB_NAME_2];
CREATE DATABASE [DB_NAME_3];

-- Extensions (if needed)
\c [DB_NAME_1]
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

**Init Script Location**: [Path to init SQL file]

### Migrations
- **Tool**: [golang-migrate / Flyway / Alembic / Prisma]
- **Location**: [Path to migrations per service]
- **Naming**: [Convention — e.g., V001__create_users_table.sql]

## Message Broker

### Configuration
- **Technology**: [Apache Kafka / RabbitMQ / NATS]
- **Host**: [BROKER_HOST]
- **Port**: [BROKER_PORT]
- **Management UI**: [URL if available]

### Topics / Queues

| Topic/Queue | Partitions | Retention | Purpose |
|------------|-----------|-----------|---------|
| [TOPIC_1] | [COUNT] | [DURATION] | [Purpose] |
| [TOPIC_2] | [COUNT] | [DURATION] | [Purpose] |
| [TOPIC_3] | [COUNT] | [DURATION] | [Purpose] |

### Consumer Groups

| Group ID | Service | Topics Consumed |
|----------|---------|----------------|
| [GROUP_1] | [SERVICE] | [TOPICS] |
| [GROUP_2] | [SERVICE] | [TOPICS] |

### Broker Dependencies (if applicable)
- **ZooKeeper**: [HOST:PORT] (for Kafka)
- **Schema Registry**: [HOST:PORT] (if using Avro/Protobuf schemas)

## API Gateway

### Configuration
- **Technology**: [Technology name]
- **Port**: [GATEWAY_PORT]
- **Config File**: [Path to gateway config]

### Route Table

| Path Prefix | Target Service | Port | Auth Required |
|------------|---------------|------|---------------|
| /api/[resource_1] | [SERVICE_1] | [PORT] | [Yes/No] |
| /api/[resource_2] | [SERVICE_2] | [PORT] | [Yes/No] |
| /api/[resource_3] | [SERVICE_3] | [PORT] | [Yes/No] |
| /ws/[resource] | [SERVICE_WS] | [PORT] | [Yes/No] |

### Gateway Features
- [ ] Rate Limiting
- [ ] CORS Configuration
- [ ] JWT Validation
- [ ] Request/Response Logging
- [ ] Circuit Breaking
- [ ] WebSocket Proxying
- [ ] SSL Termination

## CI/CD Pipeline

### Build Pipeline
```
CODE PUSH
    |
LINT & FORMAT CHECK
    |
UNIT TESTS
    |
BUILD DOCKER IMAGES
    |
INTEGRATION TESTS
    |
PUSH TO REGISTRY
    |
DEPLOY TO [ENV]
```

### Pipeline Configuration
- **Platform**: [GitHub Actions / GitLab CI / Jenkins]
- **Config File**: [Path to CI config]
- **Docker Registry**: [Registry URL]
- **Environments**: [dev, staging, production]

### Deployment Strategy
- **Method**: [Rolling / Blue-Green / Canary]
- **Rollback**: [Manual / Automatic on failure]

## Monitoring & Observability

### Metrics
- **Tool**: [Prometheus / Datadog / CloudWatch]
- **Dashboard**: [URL]
- **Key Metrics**:
  - Request rate per service
  - Error rate (4xx, 5xx)
  - Response latency (p50, p95, p99)
  - Message broker lag
  - Database connection pool usage

### Logging
- **Aggregation**: [ELK Stack / Loki / CloudWatch]
- **Format**: [JSON structured logging]
- **Correlation**: [Trace ID / Request ID header name]
- **Retention**: [Duration]

### Tracing
- **Tool**: [Jaeger / Zipkin / X-Ray / None]
- **Propagation**: [Header name for trace context]
- **Sampling Rate**: [Percentage]

### Alerting
- **Platform**: [PagerDuty / Slack / Email]
- **Critical Alerts**: [List of critical conditions]
- **Warning Alerts**: [List of warning conditions]

## Network Configuration

### Service Communication
```
[EXTERNAL CLIENT]
        |
    [LOAD BALANCER] (port 443)
        |
    [API GATEWAY] (port [GATEWAY_PORT])
        |
    [INTERNAL NETWORK]
    |       |       |
[SVC_1] [SVC_2] [SVC_3]
    |       |       |
    [MESSAGE BROKER]
        |
    [DATABASES]
```

### Ports Summary
| Port | Service/Component |
|------|------------------|
| [PORT] | [Component] |
| [PORT] | [Component] |
| [PORT] | [Component] |

### Security
- **Network Policy**: [Description]
- **Service-to-Service Auth**: [mTLS / JWT / API Key / None]
- **Secrets Management**: [Vault / K8s secrets / env files / AWS Secrets Manager]

## Local Development Setup

### Prerequisites
```bash
# Required tools
[TOOL_1] >= [VERSION]   # e.g., Docker >= 24.0
[TOOL_2] >= [VERSION]   # e.g., Go >= 1.22
[TOOL_3] >= [VERSION]   # e.g., Node.js >= 20
```

### Quick Start
```bash
# 1. Start infrastructure
docker-compose up -d [db] [broker]

# 2. Run migrations
[migration command]

# 3. Start services
docker-compose up -d

# 4. Verify
curl http://localhost:[GATEWAY_PORT]/health
```

---

**Version**: Infrastructure v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific configuration

*This file documents WHERE and HOW your microservices run. Update it when infrastructure changes.*
