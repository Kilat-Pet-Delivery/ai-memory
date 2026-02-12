# Service Registry - [PROJECT_NAME]
*Complete inventory of all microservices*

## Service Overview

**Total Services**: [TOTAL_COUNT]
**Active**: [ACTIVE_COUNT] | **Planned**: [PLANNED_COUNT] | **Deprecated**: [DEPRECATED_COUNT]

## Service Registry Table

| # | Service Name | Port | Database | Tech Stack | Responsibility | Status |
|---|-------------|------|----------|------------|----------------|--------|
| 1 | [SERVICE_1] | [PORT] | [DB_NAME] | [LANG+FRAMEWORK] | [BRIEF_RESPONSIBILITY] | Active |
| 2 | [SERVICE_2] | [PORT] | [DB_NAME] | [LANG+FRAMEWORK] | [BRIEF_RESPONSIBILITY] | Active |
| 3 | [SERVICE_3] | [PORT] | [DB_NAME] | [LANG+FRAMEWORK] | [BRIEF_RESPONSIBILITY] | Active |
| 4 | [SERVICE_4] | [PORT] | [DB_NAME] | [LANG+FRAMEWORK] | [BRIEF_RESPONSIBILITY] | Active |
| 5 | [SERVICE_5] | [PORT] | [DB_NAME] | [LANG+FRAMEWORK] | [BRIEF_RESPONSIBILITY] | Active |

## Detailed Service Profiles

---

### Service: [SERVICE_1]
**Port**: [PORT]
**Database**: [DB_NAME] ([DB_TYPE])
**Tech Stack**: [LANGUAGE] + [FRAMEWORK] + [ORM]
**Status**: Active

#### Responsibility
[Detailed description of what this service does and its bounded context]

#### Key Features
- [Feature 1]
- [Feature 2]
- [Feature 3]

#### API Endpoints (Summary)
| Method | Path | Description |
|--------|------|-------------|
| GET | /api/[resource] | [Description] |
| POST | /api/[resource] | [Description] |
| GET | /api/[resource]/:id | [Description] |

*Full endpoint details in [api-catalog.md](../catalog/api-catalog.md)*

#### Events
| Direction | Event | Topic |
|-----------|-------|-------|
| Publishes | [event.name] | [topic] |
| Consumes | [event.name] | [topic] |

*Full event details in [event-catalog.md](../catalog/event-catalog.md)*

#### Dependencies
- **Depends on**: [List of services this service calls]
- **Depended by**: [List of services that call this service]

#### Health Check
- **Endpoint**: `GET /health`
- **Response**: `{"status": "healthy", "service": "[SERVICE_1]"}`

#### Environment Variables
| Variable | Description | Default |
|----------|-------------|---------|
| PORT | Service port | [PORT] |
| DB_HOST | Database host | localhost |
| DB_PORT | Database port | [DB_PORT] |
| DB_NAME | Database name | [DB_NAME] |
| [BROKER]_BROKERS | Message broker address | localhost:[BROKER_PORT] |

---

### Service: [SERVICE_2]
**Port**: [PORT]
**Database**: [DB_NAME] ([DB_TYPE])
**Tech Stack**: [LANGUAGE] + [FRAMEWORK] + [ORM]
**Status**: Active

#### Responsibility
[Detailed description]

#### Key Features
- [Feature 1]
- [Feature 2]

#### Dependencies
- **Depends on**: [Services]
- **Depended by**: [Services]

#### Health Check
- **Endpoint**: `GET /health`

---

*Copy the detailed profile template above for each additional service.*

## Port Allocation Map

| Port Range | Purpose |
|-----------|---------|
| [PORT_RANGE_1] | [Purpose — e.g., Core business services] |
| [PORT_RANGE_2] | [Purpose — e.g., Infrastructure services] |
| [PORT_RANGE_3] | [Purpose — e.g., API gateway] |
| [PORT_RANGE_4] | [Purpose — e.g., Databases] |
| [PORT_RANGE_5] | [Purpose — e.g., Message brokers] |

## Service Dependency Summary

```
[CLIENT] --> [API_GATEWAY]
                |
    +-----------+-----------+
    |           |           |
[SERVICE_1] [SERVICE_2] [SERVICE_3]
    |           |           |
    +-----+-----+     [SERVICE_4]
          |
    [MESSAGE_BROKER]
          |
    [SERVICE_5]
```

*Detailed dependency graph in [dependency-map.md](../feature/dependency-tracker/dependency-map.md)*

## Service Status Dashboard

| Service | Health | Last Deploy | Version | Notes |
|---------|--------|-------------|---------|-------|
| [SERVICE_1] | [OK/WARN/DOWN] | [DATE] | [VERSION] | [Notes] |
| [SERVICE_2] | [OK/WARN/DOWN] | [DATE] | [VERSION] | [Notes] |
| [SERVICE_3] | [OK/WARN/DOWN] | [DATE] | [VERSION] | [Notes] |

## Adding a New Service

When a new service is created:
1. Add entry to the **Service Registry Table** above
2. Create a **Detailed Service Profile** section
3. Update **Port Allocation Map**
4. Update **Service Dependency Summary**
5. Add endpoints to `catalog/api-catalog.md`
6. Add events to `catalog/event-catalog.md`
7. Add database to `catalog/database-catalog.md`
8. Update `master-memory.md` quick reference table

---

**Version**: Service Registry v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific configuration

*This file is the single source of truth for all microservices in the system. Keep it updated!*
