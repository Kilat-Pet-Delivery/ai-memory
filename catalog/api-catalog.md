# API Catalog - Kilat Pet Delivery
*Complete reference of all REST/gRPC endpoints across all services*

## API Overview

**Total Endpoints**: 42
**API Style**: REST (JSON) + WebSocket
**Base URL**: `http://localhost:8080`
**Auth Method**: JWT Bearer (HS256)
**API Version**: v1

## Common Headers

| Header | Value | Required |
|--------|-------|----------|
| Content-Type | application/json | Yes |
| Authorization | Bearer [JWT_TOKEN] | Protected routes |
| X-Request-ID | auto-generated | Auto (middleware) |

## Common Response Format

### Success
```json
{ "success": true, "data": {}, "message": "..." }
```

### Error
```json
{ "success": false, "error": "Human readable message" }
```

### Pagination
```json
{ "success": true, "data": [], "pagination": { "page": 1, "limit": 20, "total": 100 } }
```

---

## service-identity (Port 8004) — 6 endpoints

**Middleware**: RequestID, CORS, SecurityHeaders, Logger, Recovery, RateLimit (100/min)

| # | Method | Path | Auth | Role | Description |
|---|--------|------|------|------|-------------|
| 1 | POST | /api/v1/auth/register | No | — | Register user (owner/runner/shop) |
| 2 | POST | /api/v1/auth/login | No | — | Login, returns access+refresh JWT |
| 3 | POST | /api/v1/auth/refresh | No | — | Refresh access token |
| 4 | POST | /api/v1/auth/logout | Yes | Any | Revoke refresh token |
| 5 | GET | /api/v1/auth/profile | Yes | Any | Get user profile |
| 6 | PUT | /api/v1/auth/profile | Yes | Any | Update profile |

### Register Request
```json
{ "email": "user@example.com", "password": "...", "full_name": "...", "phone": "...", "role": "owner" }
```

### Login Response
```json
{ "access_token": "eyJ...", "refresh_token": "...", "user": { "id": "uuid", "email": "...", "role": "owner" } }
```

---

## service-booking (Port 8001) — 8 endpoints

**Middleware**: Recovery, Logger, RequestID, CORS, SecurityHeaders, Auth, RequireRole

| # | Method | Path | Auth | Role | Description |
|---|--------|------|------|------|-------------|
| 1 | POST | /api/v1/bookings | Yes | Owner | Create booking |
| 2 | GET | /api/v1/bookings | Yes | Any | List bookings (role-filtered, paginated) |
| 3 | GET | /api/v1/bookings/:id | Yes | Any | Get booking detail |
| 4 | POST | /api/v1/bookings/:id/accept | Yes | Runner | Accept booking |
| 5 | POST | /api/v1/bookings/:id/pickup | Yes | Runner | Pick up pet |
| 6 | POST | /api/v1/bookings/:id/deliver | Yes | Runner | Mark delivered |
| 7 | POST | /api/v1/bookings/:id/confirm | Yes | Owner | Confirm delivery |
| 8 | POST | /api/v1/bookings/:id/cancel | Yes | Any | Cancel booking |

### Create Booking Request
```json
{
  "pet_name": "Buddy", "pet_type": "dog", "weight_kg": 5.0,
  "pickup_address": { "address": "...", "latitude": 3.1, "longitude": 101.6 },
  "dropoff_address": { "address": "...", "latitude": 3.2, "longitude": 101.7 },
  "notes": "Friendly dog"
}
```

### Booking States: `requested → accepted → in_progress → delivered → completed` (or `cancelled`)

---

## service-payment (Port 8002) — 4 endpoints

| # | Method | Path | Auth | Role | Description |
|---|--------|------|------|------|-------------|
| 1 | POST | /api/v1/payments/initiate | Yes | Owner | Initiate escrow payment |
| 2 | GET | /api/v1/payments/:id | Yes | Any | Get payment by ID |
| 3 | GET | /api/v1/payments/booking/:bookingId | Yes | Any | Get payment by booking |
| 4 | POST | /api/v1/payments/:id/refund | Yes | Admin | Refund payment |

### Escrow States: `pending → held → released/refunded` (or `failed`)

---

## service-runner (Port 8003) — 14 endpoints

### Runner Endpoints (7)

| # | Method | Path | Auth | Role | Description |
|---|--------|------|------|------|-------------|
| 1 | POST | /api/v1/runners | Yes | Runner | Register as runner (vehicle+crate) |
| 2 | GET | /api/v1/runners/me | Yes | Runner | Get my runner profile |
| 3 | POST | /api/v1/runners/me/online | Yes | Runner | Go online |
| 4 | POST | /api/v1/runners/me/offline | Yes | Runner | Go offline |
| 5 | POST | /api/v1/runners/me/location | Yes | Runner | Update GPS location |
| 6 | POST | /api/v1/runners/me/crates | Yes | Runner | Add crate specification |
| 7 | GET | /api/v1/runners/nearby | Yes | Any | Find nearby runners (?latitude, longitude, radius_km=5) |

### Pet Shop Endpoints (7)

| # | Method | Path | Auth | Role | Description |
|---|--------|------|------|------|-------------|
| 8 | GET | /api/v1/petshops | Yes | Any | List all pet shops (?category=) |
| 9 | GET | /api/v1/petshops/nearby | Yes | Any | Nearby shops (?latitude, longitude, radius_km=10) |
| 10 | GET | /api/v1/petshops/:id | Yes | Any | Get shop detail |
| 11 | GET | /api/v1/petshops/mine | Yes | Shop | Get my shops |
| 12 | POST | /api/v1/petshops | Yes | Shop | Create pet shop |
| 13 | PUT | /api/v1/petshops/:id | Yes | Shop | Update shop |
| 14 | DELETE | /api/v1/petshops/:id | Yes | Shop | Delete shop |

---

## service-tracking (Port 8005) — 3 endpoints

| # | Method | Path | Auth | Description |
|---|--------|------|------|-------------|
| 1 | GET | /api/v1/tracking/:bookingId | JWT | Get tracking data |
| 2 | GET | /api/v1/tracking/:bookingId/route | JWT | Get route as GeoJSON |
| 3 | GET | /ws/tracking/:bookingId?token=JWT | WS | Live tracking WebSocket |

### WebSocket Message Format
```json
{ "type": "location_update", "payload": { "latitude": 3.1, "longitude": 101.6, "speed_kmh": 40, "heading": 180 }, "timestamp": "ISO-8601" }
```

---

## service-notification (Port 8006) — 6 endpoints

| # | Method | Path | Auth | Description |
|---|--------|------|------|-------------|
| 1 | GET | /api/v1/notifications | JWT | List notifications (paginated) |
| 2 | GET | /api/v1/notifications/:id | JWT | Get notification detail |
| 3 | PUT | /api/v1/notifications/:id/read | JWT | Mark as read |
| 4 | GET | /api/v1/notifications/preferences | JWT | Get notification preferences |
| 5 | PUT | /api/v1/notifications/preferences | JWT | Update preferences |
| 6 | POST | /api/v1/notifications/fcm-token | JWT | Register FCM device token |

---

## api-gateway (Port 8080) — 1 endpoint + proxy

| # | Method | Path | Description |
|---|--------|------|-------------|
| 1 | GET | /health | Aggregated health check (all services) |

### Gateway Route Table

| External Path | Target | Port |
|--------------|--------|------|
| /api/v1/auth/* | service-identity | 8004 |
| /api/v1/bookings/* | service-booking | 8001 |
| /api/v1/payments/* | service-payment | 8002 |
| /api/v1/runners/* | service-runner | 8003 |
| /api/v1/petshops/* | service-runner | 8003 |
| /api/v1/tracking/* | service-tracking | 8005 |
| /api/v1/notifications/* | service-notification | 8006 |
| /ws/tracking/* | service-tracking | 8005 |

---

## Endpoint Statistics

| Service | GET | POST | PUT | DELETE | WS | Total |
|---------|-----|------|-----|--------|-----|-------|
| service-identity | 1 | 4 | 1 | 0 | 0 | 6 |
| service-booking | 2 | 6 | 0 | 0 | 0 | 8 |
| service-payment | 2 | 2 | 0 | 0 | 0 | 4 |
| service-runner | 5 | 5 | 1 | 1 | 0 | 12+2 (health) |
| service-tracking | 2 | 0 | 0 | 0 | 1 | 3 |
| service-notification | 2 | 1 | 2 | 0 | 0 | 5+1 (fcm) |
| api-gateway | 1 | 0 | 0 | 0 | 0 | 1 |
| **Total** | | | | | | **~42** |
