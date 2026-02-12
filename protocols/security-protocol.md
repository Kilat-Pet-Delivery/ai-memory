# Security Protocol - Kilat Pet Delivery
*Authentication, authorization, and security implementation*

## Security Architecture

```
[Flutter/Next.js Apps]
        |
   [API Gateway :8080] .... Rate limiting (100/min), CORS, Security Headers
        |
   [Auth Middleware] ....... JWT validation (HS256)
        |
   [RequireRole] .......... RBAC (owner, runner, admin, shop)
        |
   [Service Handler] ...... Input validation, business logic
        |
   [PostgreSQL] ........... Parameterized queries via GORM
```

## Authentication

### JWT Configuration
- **Algorithm**: HS256
- **Secret**: Environment variable (shared across services via lib-common)
- **Access Token TTL**: 15 minutes
- **Refresh Token TTL**: 7 days
- **Issuer**: service-identity
- **Claims**: sub (user ID), role (user role), exp, iat

### Token Flow
```
1. User registers/logs in → service-identity
2. Identity returns: { access_token, refresh_token, user }
3. Client stores tokens
4. Every request: Authorization: Bearer <access_token>
5. Gateway forwards to service → AuthMiddleware validates
6. Token expired? → POST /api/v1/auth/refresh with refresh_token
7. Logout → POST /api/v1/auth/logout (revokes refresh token)
```

### Password Security
- **Algorithm**: bcrypt
- **Storage**: password_hash column in users table
- **Validation**: At registration and login
- **Plain text**: NEVER stored or logged

### WebSocket Auth
- JWT passed as query parameter: `/ws/tracking/:bookingId?token=<JWT>`
- Validated on WebSocket upgrade (HTTP headers not available for WS)

## Authorization (RBAC)

### Roles
| Role | Description | Registration |
|------|-----------|--------------|
| **owner** | Pet owner — books deliveries, makes payments | POST /auth/register (role: "owner") |
| **runner** | Delivery runner — accepts jobs, provides GPS | POST /auth/register (role: "runner") |
| **shop** | Pet shop owner — manages shop listings | POST /auth/register (role: "shop") |
| **admin** | System admin — refunds, system management | Manual creation |

### Permission Matrix

| Endpoint | owner | runner | shop | admin |
|----------|-------|--------|------|-------|
| POST /bookings (create) | YES | — | — | — |
| GET /bookings (list own) | YES | YES | — | YES |
| POST /bookings/:id/accept | — | YES | — | — |
| POST /bookings/:id/pickup | — | YES | — | — |
| POST /bookings/:id/deliver | — | YES | — | — |
| POST /bookings/:id/confirm | YES | — | — | — |
| POST /bookings/:id/cancel | YES | YES | — | YES |
| POST /payments/initiate | YES | — | — | — |
| POST /payments/:id/refund | — | — | — | YES |
| POST /runners (register) | — | YES | — | — |
| GET /runners/me | — | YES | — | — |
| POST /runners/me/online | — | YES | — | — |
| GET /runners/nearby | YES | YES | YES | YES |
| GET /petshops | YES | YES | YES | YES |
| GET /petshops/mine | — | — | YES | — |
| POST /petshops | — | — | YES | — |
| PUT /petshops/:id | — | — | YES | — |
| DELETE /petshops/:id | — | — | YES | — |
| GET /tracking/:bookingId | YES | YES | — | YES |
| WS /ws/tracking/:bookingId | YES | YES | — | YES |
| GET /notifications | YES | YES | YES | YES |

### Resource Ownership
- Owners can only see their own bookings
- Runners see bookings assigned to them (or available ones)
- Shop owners can only CRUD their own shops (checked via owner_id)
- Notifications are filtered by user_id automatically

## API Gateway Security

### Middleware Stack (applied to every request)
1. **RecoveryMiddleware** — Catches panics, returns 500
2. **LoggerMiddleware** — Logs all requests with timing
3. **RequestIDMiddleware** — Adds unique X-Request-ID header
4. **CORSMiddleware** — Cross-origin resource sharing
5. **SecurityHeadersMiddleware** — X-Frame-Options, X-Content-Type-Options, etc.
6. **RateLimitMiddleware** — 100 requests per minute per IP

### Security Headers Applied
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### CORS Configuration
- Allowed origins: * (development) — restrict in production
- Allowed methods: GET, POST, PUT, DELETE, OPTIONS
- Allowed headers: Content-Type, Authorization
- Credentials: true

### Rate Limiting
- **Limit**: 100 requests per minute per IP
- **Response**: 429 Too Many Requests

## Data Security

### Sensitive Data Handling
| Data | Protection |
|------|-----------|
| Passwords | bcrypt hash, never logged |
| JWT tokens | Signed HS256, short-lived (15min) |
| Refresh tokens | Stored hashed in DB, revocable |
| GPS coordinates | Stored in PostGIS, access via auth only |
| Payment info | Mock Stripe (no real card data stored) |
| FCM tokens | Stored in preferences table, auth required |

### Database Security
- **Queries**: All via GORM ORM (parameterized, no SQL injection)
- **Connection**: Docker internal network (not exposed externally)
- **Per-service access**: Each service only accesses its own database
- **Optimistic locking**: Version column prevents concurrent update conflicts

### Event Security
- Events contain IDs, not sensitive data (no passwords, no tokens)
- Events reference users by UUID, not by email/phone
- CloudEvents envelope provides traceability (event ID, source, timestamp)

## Known Security Considerations

| Item | Status | Notes |
|------|--------|-------|
| JWT HS256 (symmetric) | Active | Consider RS256 (asymmetric) for production |
| CORS allows all origins | Active | Restrict to specific domains in production |
| No HTTPS (dev) | Expected | Add TLS termination for production |
| No API key for service-to-service | Active | Services trust Docker network |
| Mock Stripe | Active | Replace with real Stripe for production |
| No audit logging | TODO | Add security event logging |
| No brute force protection | TODO | Add account lockout after N failed logins |

## Security Checklist for Kilat

```
Authentication:
[x] JWT validation middleware on all protected routes
[x] Token expiry enforced (15min access, 7day refresh)
[x] Refresh token flow implemented
[x] Logout revokes refresh token
[x] bcrypt password hashing

Authorization:
[x] RBAC middleware (RequireRole)
[x] 4 roles: owner, runner, shop, admin
[x] Resource ownership checks (own bookings, own shops)

API Security:
[x] Rate limiting (100/min)
[x] CORS configured
[x] Security headers (XSS, clickjacking protection)
[x] Request ID for tracing

Database:
[x] GORM parameterized queries (no SQL injection)
[x] Database per service (isolation)
[x] Optimistic locking (version column)

Infrastructure:
[x] Docker internal network
[x] Services not exposed directly (gateway only)
[ ] TLS termination (TODO for production)
[ ] Secret manager (TODO — currently env vars)
[ ] Audit logging (TODO)
```
