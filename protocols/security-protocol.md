# Security Protocol - Microservices MemoryCore
*Authentication, authorization, API security, and OWASP protection for microservices*

## Why Microservices Security is Different

In a monolith, there's one security boundary. In microservices, **every service is an attack surface**. Requests cross network boundaries, services talk to each other, and data flows through message brokers. This protocol ensures security at every layer.

## Security Architecture Overview

```
[EXTERNAL CLIENT]
        |
   [WAF / CDN]  ............. Layer 1: Edge Security
        |
   [API GATEWAY]  ........... Layer 2: Gateway Security (Auth, Rate Limit, CORS)
        |
   [SERVICE MESH / mTLS] .... Layer 3: Service-to-Service Security
        |
   [MICROSERVICE]  .......... Layer 4: Application Security (Input validation, AuthZ)
        |
   [DATABASE]  .............. Layer 5: Data Security (Encryption, Access Control)
```

## Layer 1: Edge Security

### Web Application Firewall (WAF)
- **Technology**: [AWS WAF / Cloudflare / ModSecurity / None]
- **Rules**: [SQL injection, XSS, bot detection]
- **Rate Limiting**: [Global rate limits]

### TLS/SSL
- **Certificate**: [Let's Encrypt / ACM / Custom CA]
- **TLS Version**: [1.2+ minimum]
- **HSTS**: [Enabled/Disabled]

### DDoS Protection
- **Provider**: [Cloudflare / AWS Shield / None]
- **Mitigation**: [Auto / Manual]

## Layer 2: API Gateway Security

### Authentication
- **Method**: [JWT / OAuth2 / API Key]
- **Token Issuer**: [SERVICE_NAME] (e.g., identity-service)
- **Token Format**: JWT
- **Token Location**: `Authorization: Bearer <token>`
- **Token Expiry**: [ACCESS_TOKEN_TTL] (e.g., 15 minutes)
- **Refresh Token**: [REFRESH_TOKEN_TTL] (e.g., 7 days)

### JWT Configuration
```
Algorithm: [HS256 / RS256 / ES256]
Secret/Key: stored in [ENV / Vault / K8s Secret]
Claims:
  - sub: user ID
  - role: user role
  - exp: expiration timestamp
  - iat: issued at
  - iss: issuer service name
```

### Token Validation Flow
```
CLIENT sends request with Authorization header
        |
API GATEWAY extracts JWT
        |
VALIDATE: signature, expiration, issuer
        |
    VALID? -- NO --> 401 Unauthorized
        |
       YES
        |
FORWARD to target service with user context
```

### Rate Limiting
| Endpoint Type | Limit | Window | Response |
|--------------|-------|--------|----------|
| Public (login, register) | [N] req | [WINDOW] | 429 Too Many Requests |
| Authenticated | [N] req | [WINDOW] | 429 Too Many Requests |
| Admin | [N] req | [WINDOW] | 429 Too Many Requests |
| Health checks | Unlimited | — | — |

### CORS Configuration
```
Allowed Origins: [LIST_OF_ORIGINS]
Allowed Methods: GET, POST, PUT, DELETE, OPTIONS
Allowed Headers: Content-Type, Authorization, X-Request-ID
Expose Headers: X-Request-ID
Max Age: [SECONDS]
Credentials: [true/false]
```

## Layer 3: Service-to-Service Security

### Internal Communication
- **Method**: [mTLS / JWT propagation / API Key / Network policy]
- **Trust Model**: [Zero trust / Network-level trust]

### mTLS (If Used)
```
Each service has:
  - Client certificate (to prove identity)
  - CA certificate (to verify other services)
  - Private key (for encryption)

Certificate Management: [cert-manager / Vault / manual]
Rotation: [Automatic / Manual, every N days]
```

### JWT Propagation (If Used)
```
Gateway validates JWT and forwards to downstream services.
Each service:
  1. Extracts JWT from Authorization header
  2. Validates signature (shared secret or public key)
  3. Extracts user ID and roles from claims
  4. Applies authorization rules
```

### Service Identity
| Service | Identity Method | Credentials |
|---------|---------------|------------|
| [SERVICE_1] | [mTLS cert / API key / JWT] | [Where stored] |
| [SERVICE_2] | [mTLS cert / API key / JWT] | [Where stored] |

## Layer 4: Application Security

### Authentication Middleware
```
Every protected endpoint:
  1. Extract token from Authorization header
  2. Validate token (signature, expiry, issuer)
  3. Extract user claims (ID, role)
  4. Attach user context to request
  5. Proceed to handler
```

### Authorization (RBAC)
**Roles**:
| Role | Description | Access Level |
|------|-----------|-------------|
| [ROLE_1] | [Description] | [What they can access] |
| [ROLE_2] | [Description] | [What they can access] |
| [ROLE_3] | [Description] | [What they can access] |
| [ADMIN] | System administrator | Full access |

**Permission Matrix**:
| Endpoint | [ROLE_1] | [ROLE_2] | [ROLE_3] | [ADMIN] |
|----------|----------|----------|----------|---------|
| GET /api/[resource] | [Y/N] | [Y/N] | [Y/N] | Y |
| POST /api/[resource] | [Y/N] | [Y/N] | [Y/N] | Y |
| PUT /api/[resource]/:id | [Own/All/N] | [Own/All/N] | [Y/N] | Y |
| DELETE /api/[resource]/:id | [N] | [N] | [Y/N] | Y |

### Input Validation

**Rules (Apply to ALL endpoints)**:
```
[ ] Validate all request body fields (type, length, format)
[ ] Validate path parameters (UUID format, numeric range)
[ ] Validate query parameters (allowed values, pagination limits)
[ ] Sanitize string inputs (trim whitespace, escape special chars)
[ ] Reject unexpected fields (strict schema validation)
[ ] Validate Content-Type header
[ ] Limit request body size
```

**Validation Pattern**:
```
REQUEST RECEIVED
      |
PARSE JSON (reject malformed)
      |
VALIDATE SCHEMA (required fields, types)
      |
VALIDATE BUSINESS RULES (ranges, formats)
      |
SANITIZE (trim, escape)
      |
PROCESS
```

### OWASP Top 10 Protection

| # | Vulnerability | Protection | Implementation |
|---|-------------|-----------|----------------|
| 1 | **Broken Access Control** | RBAC + resource ownership checks | Middleware checks role AND ownership |
| 2 | **Cryptographic Failures** | Hash passwords, encrypt sensitive data | bcrypt for passwords, AES for data |
| 3 | **Injection** | Parameterized queries, ORM | Never concatenate SQL strings |
| 4 | **Insecure Design** | Threat modeling, DDD boundaries | Bounded contexts enforce isolation |
| 5 | **Security Misconfiguration** | Hardened defaults, no debug in prod | Environment-specific configs |
| 6 | **Vulnerable Components** | Dependency scanning | Dependabot / Snyk / govulncheck |
| 7 | **Auth Failures** | JWT best practices, token rotation | Short-lived tokens, refresh flow |
| 8 | **Data Integrity Failures** | Signed events, schema validation | CloudEvents + schema registry |
| 9 | **Logging & Monitoring** | Structured logging, audit trail | Log auth events, access patterns |
| 10 | **SSRF** | Whitelist allowed internal URLs | Validate outbound request targets |

### Password Security
```
Algorithm: bcrypt (cost factor [N])
Minimum Length: [N] characters
Requirements: [uppercase, lowercase, number, special]
Storage: NEVER plaintext, ALWAYS hashed
Reset Flow: [Token-based email reset]
```

### SQL Injection Prevention
```
NEVER:
  query := "SELECT * FROM users WHERE id = '" + userInput + "'"

ALWAYS:
  db.Where("id = ?", userInput).First(&user)           // ORM
  db.Query("SELECT * FROM users WHERE id = $1", userID) // Parameterized
```

### XSS Prevention
```
[ ] Sanitize all user input before storage
[ ] Encode output in HTML responses
[ ] Set Content-Type headers correctly
[ ] Use CSP (Content-Security-Policy) headers
[ ] Validate file uploads (type, size, content)
```

## Layer 5: Data Security

### Encryption
| Data Type | At Rest | In Transit | Method |
|-----------|---------|-----------|--------|
| Passwords | bcrypt hash | TLS | bcrypt cost [N] |
| PII (name, email) | [Encrypted/Plain] | TLS | [AES-256 / Plain] |
| Payment data | Encrypted | TLS | [PCI-DSS compliant] |
| Session tokens | — | TLS | JWT signed with [ALG] |
| Database | [TDE / None] | [SSL / None] | [Method] |

### Data Classification
| Classification | Examples | Protection |
|---------------|----------|-----------|
| **Public** | Product names, prices | No special protection |
| **Internal** | User IDs, order statuses | Auth required |
| **Confidential** | Email, phone, address | Auth + encryption + audit |
| **Secret** | Passwords, payment info, API keys | Hash/encrypt + strict access + audit |

### Secrets Management
| Secret | Storage | Rotation | Access |
|--------|---------|---------|--------|
| JWT signing key | [Vault / ENV / K8s Secret] | [Frequency] | [Auth service only] |
| Database passwords | [Vault / ENV / K8s Secret] | [Frequency] | [Per-service] |
| API keys (3rd party) | [Vault / ENV / K8s Secret] | [Frequency] | [Relevant service] |
| Broker credentials | [Vault / ENV / K8s Secret] | [Frequency] | [All services] |

**Rules**:
```
[ ] NEVER commit secrets to git
[ ] NEVER log secrets or tokens
[ ] NEVER hardcode credentials in source code
[ ] USE environment variables or secret managers
[ ] ROTATE secrets on regular schedule
[ ] REVOKE compromised secrets immediately
```

## Event Security

### Message Broker Security
- **Authentication**: [SASL / mTLS / None]
- **Authorization**: [ACL per topic / None]
- **Encryption**: [TLS in transit / None]
- **Schema Validation**: [Schema Registry / Application-level]

### Event Integrity
```
[ ] Validate event schema before processing
[ ] Check event source (is the producer authorized?)
[ ] Handle duplicate events (idempotency)
[ ] Reject events with expired timestamps
[ ] Log all consumed events for audit trail
[ ] Protect against poison messages (DLQ)
```

### Sensitive Data in Events
```
NEVER put in events:
  - Passwords or hashes
  - Full credit card numbers
  - API keys or tokens

USE instead:
  - Reference IDs (userId instead of full user object)
  - Masked data (card ending in ****1234)
  - Separate encrypted channel for sensitive data
```

## Security Monitoring & Audit

### Security Events to Log
| Event | Severity | What to Log |
|-------|----------|-------------|
| Failed login | WARN | IP, email, timestamp |
| Successful login | INFO | UserID, IP, timestamp |
| Token expired / invalid | WARN | Token claims, IP |
| Authorization denied | WARN | UserID, resource, action |
| Rate limit hit | WARN | IP, endpoint, count |
| Input validation failed | INFO | Endpoint, field, reason |
| Admin action | INFO | AdminID, action, target |
| Data export / access | INFO | UserID, data type, count |

### Audit Log Format
```json
{
  "timestamp": "ISO-8601",
  "level": "WARN",
  "event": "auth.login.failed",
  "service": "[SERVICE_NAME]",
  "ip": "192.168.1.1",
  "user_agent": "...",
  "details": {
    "email": "user@example.com",
    "reason": "invalid_password"
  }
}
```

### Security Alerts
| Condition | Action |
|-----------|--------|
| 5+ failed logins from same IP in 5 min | Temporary IP block |
| 10+ failed logins for same account in 1 hour | Account lockout |
| Token used after revocation | Alert + investigate |
| SQL injection attempt detected | Log + block + alert |
| Unusual data access pattern | Alert for review |

## Security Checklist for New Service

```
Authentication:
[ ] JWT validation middleware applied to protected routes
[ ] Token expiry enforced
[ ] Refresh token flow implemented (if needed)

Authorization:
[ ] RBAC middleware applied
[ ] Resource ownership checks (users can only access own data)
[ ] Admin-only endpoints protected

Input Validation:
[ ] All request bodies validated against schema
[ ] Path parameters validated
[ ] Query parameters validated
[ ] File uploads validated (type, size)
[ ] Request size limits set

Data Protection:
[ ] Passwords hashed with bcrypt
[ ] Sensitive data encrypted at rest (if required)
[ ] PII not logged
[ ] Secrets in environment variables, not code

API Security:
[ ] CORS configured correctly
[ ] Rate limiting applied
[ ] Request/response headers secure
[ ] Error messages don't leak internal details

Database:
[ ] Parameterized queries / ORM used (no SQL injection)
[ ] Database user has minimum required permissions
[ ] Connection uses SSL (production)

Logging:
[ ] Auth events logged
[ ] Security events logged
[ ] No secrets or PII in logs

Dependencies:
[ ] No known vulnerabilities in dependencies
[ ] Dependency scanning enabled
```

## Common Security Mistakes in Microservices

| Mistake | Risk | Fix |
|---------|------|-----|
| Trusting internal network | Lateral movement after breach | Zero trust, validate everything |
| Shared database credentials | One compromised service = all data | Per-service DB credentials |
| Logging JWT tokens | Token theft from logs | Log only claims, never full token |
| No rate limiting | Brute force, DDoS | Rate limit at gateway |
| Hardcoded secrets | Secrets in git history | Use secret manager |
| Over-permissive CORS | Cross-origin attacks | Whitelist specific origins |
| No input validation | Injection attacks | Validate everything |
| Admin endpoints exposed | Unauthorized access | Separate admin routes, IP whitelist |
| No audit logging | Can't detect or investigate breaches | Log all security events |
| Same JWT secret everywhere | One leak compromises all services | Per-service keys or asymmetric (RS256) |

---

**Version**: Security Protocol v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific security configuration

*This file ensures your microservices are secure at every layer. Review it before every deployment!*
