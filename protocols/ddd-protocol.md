# DDD Protocol - Kilat Pet Delivery
*Bounded contexts, aggregates, domain events for the Kilat system*

## Strategic Design

### Bounded Contexts

| # | Bounded Context | Service | Domain Type | Description |
|---|----------------|---------|-------------|-------------|
| 1 | **Identity** | service-identity | Generic | Auth, JWT tokens, user management, roles |
| 2 | **Booking** | service-booking | Core | Booking lifecycle, pricing, state machine |
| 3 | **Payment** | service-payment | Core | Escrow, Saga pattern, financial transactions |
| 4 | **Runner** | service-runner | Supporting | Runner profiles, geospatial, pet shops |
| 5 | **Tracking** | service-tracking | Core | Real-time GPS tracking, waypoints, WebSocket |
| 6 | **Notification** | service-notification | Generic | Multi-channel delivery (FCM, SMS, email) |

### Domain Classification

| Type | Contexts | Investment |
|------|----------|-----------|
| **Core** | Booking, Payment, Tracking | High — unique to pet delivery, competitive advantage |
| **Supporting** | Runner | Medium — supports core but not unique (driver management) |
| **Generic** | Identity, Notification | Low — standard auth/notification, could use off-the-shelf |

### Context Map

```
[Identity] ──Open Host Service──→ [All Services]   (JWT validation)
     |
[Booking] ←──Partnership──→ [Payment]              (Saga: escrow lifecycle)
     |              |
     +──Published Language──→ [Tracking]            (booking.accepted → create trip)
     |              |
     +──Published Language──→ [Notification]        (all events → send notifications)
     |
[Runner] ──Published Language──→ [Tracking]         (location updates → waypoints)
     |
[Tracking] ──Published Language──→ [Notification]   (tracking events)
```

### Context Relationships

| Upstream | Downstream | Relationship | How |
|----------|-----------|-------------|-----|
| Identity | All services | Open Host Service | JWT tokens validated by all services via shared middleware |
| Booking | Payment | Partnership | Saga: delivery_confirmed → release escrow, cancelled → refund |
| Booking | Tracking | Published Language | booking.accepted → create TripTrack |
| Booking | Notification | Published Language | All 7 booking events → notifications |
| Payment | Booking | Published Language | escrow_released → complete booking |
| Payment | Notification | Published Language | Payment events → notifications |
| Runner | Tracking | Published Language | location_update → add waypoint |
| Tracking | Notification | Published Language | Tracking events → notifications |

## Tactical Design

### Aggregates

| Bounded Context | Aggregate Root | Contains | References (by ID) |
|----------------|---------------|----------|-------------------|
| Identity | **User** | RefreshToken | — |
| Booking | **Booking** | PetSpecification (VO), CrateRequirement (VO), RouteSpecification (VO), AddressDTO (VO) | ownerID → User, runnerID → Runner |
| Payment | **Payment** | — | bookingID → Booking, ownerID → User, runnerID → Runner |
| Runner | **Runner** | — | userID → User |
| Runner | **PetShop** | — | ownerID → User |
| Tracking | **TripTrack** | Waypoint (VO) | bookingID → Booking, runnerID → Runner |
| Notification | **Notification** | — | userID → User, bookingID → Booking |
| Notification | **NotificationPreference** | — | userID → User |

### Entities vs Value Objects

**Entities** (have identity, mutable):
| Entity | Context | Identity | Key Behavior |
|--------|---------|----------|-------------|
| User | Identity | UUID | Register, Login, UpdateProfile |
| RefreshToken | Identity | UUID | Create, Revoke, CheckExpiry |
| Booking | Booking | UUID + BookingNumber | Accept, Pickup, Deliver, Confirm, Cancel (state machine) |
| Payment | Payment | UUID | InitiateEscrow, HoldEscrow, ReleaseEscrow, RefundEscrow (Saga) |
| Runner | Runner | UUID | GoOnline, GoOffline, UpdateLocation |
| PetShop | Runner | UUID | Create, Update, Delete |
| TripTrack | Tracking | UUID | Start, AddWaypoint, Complete |
| Notification | Notification | UUID | Send, MarkRead, Retry |
| NotificationPreference | Notification | UUID | Update preferences |

**Value Objects** (no identity, immutable):
| Value Object | Context | Attributes |
|-------------|---------|-----------|
| PetSpecification | Booking | name, petType, weightKg, age, allergies, notes |
| CrateRequirement | Booking | size, type |
| RouteSpecification | Booking | distance, duration, polyline |
| AddressDTO | Booking | address, latitude, longitude |
| Waypoint | Tracking | latitude, longitude, speed, heading, recordedAt |

### Domain Events (19 total)

| Context | Events Published | Count |
|---------|-----------------|-------|
| Booking | requested, accepted, pet_picked_up, delivery_in_progress, delivery_confirmed, completed, cancelled | 7 |
| Payment | escrow_created, escrow_held, escrow_released, escrow_refunded, failed | 5 |
| Runner | online, offline, location_update, assigned | 4 |
| Tracking | started, updated, completed | 3 |

### Domain Services

| Context | Domain Service | Purpose | Aggregates Used |
|---------|---------------|---------|-----------------|
| Booking | BookingService | State transitions, pricing, validation | Booking |
| Payment | PaymentService | Escrow Saga (hold, release, refund) | Payment |
| Runner | RunnerService | Online/offline, location, nearby search | Runner |
| Runner | PetShopService | CRUD, nearby search | PetShop |
| Tracking | TrackingService | Trip lifecycle, waypoints, WebSocket hub | TripTrack |
| Notification | NotificationService | Multi-channel dispatch | Notification, NotificationPreference |

### Repositories (one per Aggregate Root)

| Repository Interface | Context | Implementation |
|---------------------|---------|----------------|
| UserRepository | Identity | PostgresUserRepository (GORM) |
| TokenRepository | Identity | PostgresTokenRepository (GORM) |
| BookingRepository | Booking | PostgresBookingRepository (GORM) |
| PaymentRepository | Payment | PostgresPaymentRepository (GORM) |
| RunnerRepository | Runner | PostgresRunnerRepository (GORM + PostGIS) |
| PetShopRepository | Runner | PostgresPetShopRepository (GORM + PostGIS) |
| TrackingRepository | Tracking | PostgresTrackingRepository (GORM + PostGIS) |
| NotificationRepository | Notification | PostgresNotificationRepository (GORM) |
| PreferenceRepository | Notification | PostgresPreferenceRepository (GORM) |

## Ubiquitous Language

### Booking Context
| Term | Definition |
|------|-----------|
| Booking | A pet delivery request from owner to runner |
| BookingNumber | Unique human-readable ID (BK-XXXXXX) |
| PetSpec | Pet details (name, type, weight, allergies) |
| Pickup/Dropoff | Source and destination addresses |
| Status | State machine: requested → accepted → in_progress → delivered → completed |

### Payment Context
| Term | Definition |
|------|-----------|
| Escrow | Funds held by platform during delivery |
| Hold | Freeze funds from owner's payment method |
| Release | Transfer held funds to runner after delivery |
| Refund | Return held funds to owner on cancellation |
| Platform Fee | Commission taken by Kilat platform |
| Payout | Amount runner receives (total - platform fee) |

### Runner Context
| Term | Definition |
|------|-----------|
| Runner | A delivery person who transports pets |
| Session Status | Whether runner is active (online) or inactive (offline) |
| Crate | Pet transport container specification |
| Nearby | Within radius (default 5km for runners, 10km for shops) |

### Tracking Context
| Term | Definition |
|------|-----------|
| TripTrack | A GPS tracking session for one booking |
| Waypoint | A single GPS coordinate point along the route |
| GeoJSON | Standard format for representing the route on a map |
| Hub | WebSocket connection manager for live updates |
