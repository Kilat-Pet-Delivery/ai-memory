# API Catalog - [PROJECT_NAME]
*Complete reference of all REST/gRPC endpoints across all services*

## API Overview

**Total Endpoints**: [TOTAL_COUNT]
**API Style**: [REST / gRPC / GraphQL]
**Base URL**: `http://localhost:[GATEWAY_PORT]`
**Auth Method**: [JWT Bearer / API Key / OAuth2]
**API Version**: [v1 / v2]

## Common Headers

| Header | Value | Required | Description |
|--------|-------|----------|-------------|
| Content-Type | application/json | Yes | Request body format |
| Authorization | Bearer [JWT_TOKEN] | Conditional | Auth token for protected routes |
| X-Request-ID | [UUID] | No | Request tracing ID |
| X-Correlation-ID | [UUID] | No | Cross-service correlation |

## Common Response Format

### Success Response
```json
{
  "success": true,
  "data": {},
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "details": {}
  }
}
```

### Pagination Response
```json
{
  "success": true,
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

---

## [SERVICE_1] Endpoints

**Base Path**: `/api/v1/[resource_1]`
**Service Port**: [PORT]
**Auth**: [Required / Optional / None]

| # | Method | Path | Auth | Description |
|---|--------|------|------|-------------|
| 1 | GET | /api/v1/[resource] | [Yes/No] | List all [resources] |
| 2 | POST | /api/v1/[resource] | [Yes/No] | Create new [resource] |
| 3 | GET | /api/v1/[resource]/:id | [Yes/No] | Get [resource] by ID |
| 4 | PUT | /api/v1/[resource]/:id | [Yes/No] | Update [resource] |
| 5 | DELETE | /api/v1/[resource]/:id | [Yes/No] | Delete [resource] |

### Endpoint Details

#### `POST /api/v1/[resource]` — Create [Resource]
**Auth**: Required
**Request Body**:
```json
{
  "field_1": "string",
  "field_2": 123,
  "field_3": true
}
```
**Response** (201):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "field_1": "string",
    "field_2": 123,
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```
**Errors**:
| Code | Status | Description |
|------|--------|-------------|
| VALIDATION_ERROR | 400 | Invalid request body |
| UNAUTHORIZED | 401 | Missing or invalid token |
| CONFLICT | 409 | Resource already exists |

---

## [SERVICE_2] Endpoints

**Base Path**: `/api/v1/[resource_2]`
**Service Port**: [PORT]
**Auth**: [Required / Optional / None]

| # | Method | Path | Auth | Description |
|---|--------|------|------|-------------|
| 1 | GET | /api/v1/[resource] | [Yes/No] | [Description] |
| 2 | POST | /api/v1/[resource] | [Yes/No] | [Description] |

---

## [API_GATEWAY] Endpoints

**Port**: [GATEWAY_PORT]

### Health & System
| Method | Path | Description |
|--------|------|-------------|
| GET | /health | Gateway health check |
| GET | /api/v1/health | Aggregate health of all services |

### Route Mapping
| External Path | Internal Service | Internal Port |
|--------------|-----------------|---------------|
| /api/v1/[resource_1]/* | [SERVICE_1] | [PORT] |
| /api/v1/[resource_2]/* | [SERVICE_2] | [PORT] |
| /api/v1/[resource_3]/* | [SERVICE_3] | [PORT] |
| /ws/* | [WS_SERVICE] | [PORT] |

---

## Authentication Endpoints

*If auth is handled by a dedicated service:*

| # | Method | Path | Auth | Description |
|---|--------|------|------|-------------|
| 1 | POST | /api/v1/auth/register | No | User registration |
| 2 | POST | /api/v1/auth/login | No | User login, returns JWT |
| 3 | POST | /api/v1/auth/refresh | Yes | Refresh JWT token |
| 4 | POST | /api/v1/auth/logout | Yes | Invalidate token |
| 5 | GET | /api/v1/auth/me | Yes | Get current user profile |

---

## WebSocket Endpoints

| Path | Service | Protocol | Auth | Description |
|------|---------|----------|------|-------------|
| /ws/[channel] | [SERVICE] | WebSocket | [Yes/No] | [Description] |

### WebSocket Message Format
```json
{
  "type": "[message_type]",
  "payload": {},
  "timestamp": "ISO-8601"
}
```

---

## Endpoint Statistics

| Service | GET | POST | PUT | DELETE | Total |
|---------|-----|------|-----|--------|-------|
| [SERVICE_1] | [N] | [N] | [N] | [N] | [N] |
| [SERVICE_2] | [N] | [N] | [N] | [N] | [N] |
| [SERVICE_3] | [N] | [N] | [N] | [N] | [N] |
| **Total** | **[N]** | **[N]** | **[N]** | **[N]** | **[N]** |

## Adding a New Endpoint

When a new endpoint is created:
1. Add entry to the service's endpoint table above
2. Add detailed request/response documentation
3. Update endpoint statistics
4. Update gateway route mapping if needed
5. Note auth requirements

---

**Version**: API Catalog v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific endpoints

*This file is the single source of truth for all API endpoints. Keep it updated when endpoints change!*
