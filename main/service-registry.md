# Service Registry - Kilat Pet Delivery
*Complete inventory of all microservices*

## Service Overview

**Total Services**: 8 (7 business + 1 gateway)
**Active**: 8 | **Planned**: 0 | **Deprecated**: 0

## Service Registry Table

| # | Service | Port | Database | Tech Stack | Responsibility | Status |
|---|---------|------|----------|------------|----------------|--------|
| 1 | service-identity | 8004 | kilat_identity | Go+Gin+GORM | Auth, JWT, roles, referrals, admin users | Active |
| 2 | service-booking | 8001 | kilat_booking | Go+Gin+GORM | Booking, pet profiles, photos, admin bookings | Active |
| 3 | service-payment | 8002 | kilat_payment | Go+Gin+GORM | Escrow, promos, subscriptions, admin payments | Active |
| 4 | service-runner | 8003 | kilat_runner (PostGIS) | Go+Gin+GORM | Runner profiles, nearby search, pet shops | Active |
| 5 | service-tracking | 8005 | kilat_tracking (PostGIS) | Go+Gin+GORM | WebSocket hub, GPS, chat, trip sharing | Active |
| 6 | service-notification | 8006 | kilat_notification | Go+Gin+GORM | FCM push, Twilio SMS, SMTP email | Active |
| 7 | service-review | 8007 | kilat_review | Go+Gin+GORM | Rating & reviews (NEW Feb 2026) | Active |
| 8 | api-gateway | 8080 | — | Go+Gin | Reverse proxy, WS tunneling, rate limit | Active |

---

### Service: service-identity
**Port**: 8004 | **Database**: kilat_identity (uuid-ossp) | **Status**: Active

**Responsibility**: Authentication and user management. Issues JWT tokens, manages user registration/login, profile CRUD, and role-based access (Owner, Runner, Admin, Shop).

**Key Features**: JWT access+refresh tokens, bcrypt passwords, role-based registration, profile management, optimistic locking

**API Summary**:
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/v1/auth/register | No | Register user (with role) |
| POST | /api/v1/auth/login | No | Login, returns JWT |
| POST | /api/v1/auth/refresh | No | Refresh access token |
| POST | /api/v1/auth/logout | Yes | Revoke refresh token |
| GET | /api/v1/auth/profile | Yes | Get user profile |
| PUT | /api/v1/auth/profile | Yes | Update profile |

**Events**: None (auth is synchronous)
**Dependencies**: None (standalone)
**Depended By**: All services (JWT validation)

---

### Service: service-booking
**Port**: 8001 | **Database**: kilat_booking (PostGIS, uuid-ossp) | **Status**: Active

**Responsibility**: Booking lifecycle management. State machine: requested → accepted → in_progress → delivered → completed (or cancelled). Pricing calculation, pet specifications.

**Key Features**: Booking state machine, JSONB storage (pet spec, addresses, route), pagination, optimistic locking

**API Summary**:
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/v1/bookings | Owner | Create booking |
| GET | /api/v1/bookings | Any | List bookings (role-filtered) |
| GET | /api/v1/bookings/:id | Any | Get booking detail |
| POST | /api/v1/bookings/:id/accept | Runner | Accept booking |
| POST | /api/v1/bookings/:id/pickup | Runner | Pick up pet |
| POST | /api/v1/bookings/:id/deliver | Runner | Mark delivered |
| POST | /api/v1/bookings/:id/confirm | Owner | Confirm delivery |
| POST | /api/v1/bookings/:id/cancel | Any | Cancel booking |

**Events Published**: booking.requested, booking.accepted, booking.pet_picked_up, booking.delivery_in_progress, booking.delivery_confirmed, booking.completed, booking.cancelled
**Events Consumed**: payment.escrow_released (→ completes booking)
**Dependencies**: service-payment (via events)

---

### Service: service-payment
**Port**: 8002 | **Database**: kilat_payment (uuid-ossp) | **Status**: Active

**Responsibility**: Payment processing with escrow pattern. Holds funds during delivery, releases to runner on completion, refunds on cancellation. Mock Stripe integration. Saga pattern for distributed transactions.

**Key Features**: Escrow state machine (pending→held→released/refunded), platform fee calculation, Saga with compensation

**API Summary**:
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/v1/payments/initiate | Owner | Initiate payment for booking |
| GET | /api/v1/payments/:id | Any | Get payment by ID |
| GET | /api/v1/payments/booking/:bookingId | Any | Get payment by booking |
| POST | /api/v1/payments/:id/refund | Admin | Refund payment |

**Events Published**: payment.escrow_created, payment.escrow_held, payment.escrow_released, payment.escrow_refunded, payment.failed
**Events Consumed**: booking.delivery_confirmed (→ release escrow), booking.cancelled (→ refund)
**Dependencies**: service-booking (via events), mock Stripe

---

### Service: service-runner
**Port**: 8003 | **Database**: kilat_runner (PostGIS, uuid-ossp) | **Status**: Active

**Responsibility**: Runner profile management and geospatial features. Online/offline toggling, GPS location updates, nearby runner search. Also hosts pet shop CRUD and nearby shop search.

**Key Features**: PostGIS ST_DWithin/ST_Distance for nearby search, vehicle+crate specs, pet shop directory with owner CRUD

**API Summary**:
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/v1/runners | Runner | Register as runner |
| GET | /api/v1/runners/me | Runner | Get my profile |
| POST | /api/v1/runners/me/online | Runner | Go online |
| POST | /api/v1/runners/me/offline | Runner | Go offline |
| POST | /api/v1/runners/me/location | Runner | Update GPS location |
| POST | /api/v1/runners/me/crates | Runner | Add crate specs |
| GET | /api/v1/runners/nearby | Any | Find nearby runners |
| GET | /api/v1/petshops | Any | List pet shops |
| GET | /api/v1/petshops/nearby | Any | Find nearby shops |
| GET | /api/v1/petshops/:id | Any | Get shop detail |
| GET | /api/v1/petshops/mine | Shop | My shops |
| POST | /api/v1/petshops | Shop | Create shop |
| PUT | /api/v1/petshops/:id | Shop | Update shop |
| DELETE | /api/v1/petshops/:id | Shop | Delete shop |

**Events Published**: runner.online, runner.offline, runner.location_update
**Events Consumed**: None
**Dependencies**: None

---

### Service: service-tracking
**Port**: 8005 | **Database**: kilat_tracking (PostGIS, uuid-ossp) | **Status**: Active

**Responsibility**: Real-time delivery tracking. WebSocket hub for live GPS updates, trip track management, waypoint recording, GeoJSON route generation.

**Key Features**: WebSocket hub with JWT auth (query param), GPS waypoint recording, GeoJSON LineString generation, PostGIS

**API Summary**:
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | /api/v1/tracking/:bookingId | JWT | Get tracking data |
| GET | /api/v1/tracking/:bookingId/route | JWT | Get route as GeoJSON |
| GET | /ws/tracking/:bookingId?token=JWT | WS | Live tracking WebSocket |

**Events Published**: tracking.started, tracking.updated, tracking.completed
**Events Consumed**: booking.accepted (→ create trip track), booking.delivery_confirmed (→ complete tracking), runner.location_update (→ add waypoint + broadcast WS)
**Dependencies**: service-booking, service-runner (via events)

---

### Service: service-notification
**Port**: 8006 | **Database**: kilat_notification (uuid-ossp) | **Status**: Active

**Responsibility**: Multi-channel notification delivery. FCM push notifications, Twilio SMS, SMTP email. User preference management, read/unread tracking.

**Key Features**: Multi-channel (push/SMS/email), notification preferences, FCM token registration, quiet hours

**API Summary**:
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | /api/v1/notifications | JWT | List notifications (paginated) |
| GET | /api/v1/notifications/:id | JWT | Get notification |
| PUT | /api/v1/notifications/:id/read | JWT | Mark as read |
| GET | /api/v1/notifications/preferences | JWT | Get preferences |
| PUT | /api/v1/notifications/preferences | JWT | Update preferences |
| POST | /api/v1/notifications/fcm-token | JWT | Register FCM token |

**Events Published**: None
**Events Consumed**: ALL booking events (7), payment events (4), tracking events (2) → sends notifications
**Dependencies**: All services (consumes their events)

---

### Service: api-gateway
**Port**: 8080 | **Database**: None | **Status**: Active

**Responsibility**: Single entry point for all clients. Path-based reverse proxy, WebSocket tunneling, aggregated health check, rate limiting (100 req/min).

**Route Table**:
| Path Prefix | Target Service | Port |
|------------|---------------|------|
| /api/v1/auth/* | service-identity | 8004 |
| /api/v1/bookings/* | service-booking | 8001 |
| /api/v1/payments/* | service-payment | 8002 |
| /api/v1/runners/* | service-runner | 8003 |
| /api/v1/petshops/* | service-runner | 8003 |
| /api/v1/tracking/* | service-tracking | 8005 |
| /api/v1/notifications/* | service-notification | 8006 |
| /api/v1/reviews/* | service-review | 8007 |
| /api/v1/chat/* | service-tracking | 8005 |
| /api/v1/pets/* | service-booking | 8001 |
| /api/v1/referrals/* | service-identity | 8004 |
| /api/v1/promos/* | service-payment | 8002 |
| /api/v1/subscriptions/* | service-payment | 8002 |
| /api/v1/admin/users/* | service-identity | 8004 |
| /api/v1/admin/bookings/* | service-booking | 8001 |
| /api/v1/admin/payments/* | service-payment | 8002 |
| /ws/tracking/* | service-tracking | 8005 |

---

## Port Allocation Map

| Port | Component |
|------|-----------|
| 8001 | service-booking |
| 8002 | service-payment |
| 8003 | service-runner |
| 8004 | service-identity |
| 8005 | service-tracking |
| 8006 | service-notification |
| 8007 | service-review (NEW) |
| 8080 | api-gateway |
| 5433 | PostgreSQL 16 + PostGIS |
| 9092 | Apache Kafka |
| 2181 | Zookeeper |

## Service Dependency Graph

```
[Flutter Apps / Next.js Web] --> [API Gateway :8080]
                                      |
     +--------+--------+------+------+------+--------+
     |        |        |      |      |      |        |
 [Identity] [Booking] [Pay] [Runner] [Notif] [Review] [Tracking]
   :8004     :8001   :8002  :8003   :8006   :8007    :8005
                |       |      |       |       |        |
                +---[Kafka]----+-------+-------+--------+
```
