# Database Catalog - Kilat Pet Delivery
*Complete reference of all databases and schemas*

## Database Overview

**Primary Database**: PostgreSQL 16 + PostGIS 3.4
**Host**: localhost:5433 (host) / postgres:5432 (Docker)
**Total Databases**: 7
**Total Tables**: 20+
**ORM**: GORM (AutoMigrate)
**Migration Tool**: GORM AutoMigrate (no versioned files yet)

## Database Inventory

| # | Database | Owner Service | Extensions | Tables |
|---|---------|--------------|------------|--------|
| 1 | kilat_identity | service-identity | uuid-ossp | users, refresh_tokens, referrals, user_referral_codes |
| 2 | kilat_booking | service-booking | PostGIS, uuid-ossp | bookings, pets, booking_photos |
| 3 | kilat_payment | service-payment | uuid-ossp | payments, promos, promo_usages, subscriptions |
| 4 | kilat_runner | service-runner | PostGIS, uuid-ossp | runners, pet_shops |
| 5 | kilat_tracking | service-tracking | PostGIS, uuid-ossp | trip_tracks, waypoints, chat_messages, shared_trips |
| 6 | kilat_notification | service-notification | uuid-ossp | notifications, notification_preferences |
| 7 | **kilat_review** | **service-review** | uuid-ossp | **reviews** (NEW) |

### New Tables (Feb 2026)
| Table | Database | Feature |
|-------|----------|---------|
| pets | kilat_booking | Pet Profiles |
| booking_photos | kilat_booking | Photo Proof |
| reviews | kilat_review | Rating & Review |
| chat_messages | kilat_tracking | In-App Chat |
| shared_trips | kilat_tracking | Trip Sharing |
| promos | kilat_payment | Promo Codes |
| promo_usages | kilat_payment | Promo Codes |
| referrals | kilat_identity | Referral Program |
| user_referral_codes | kilat_identity | Referral Program |
| subscriptions | kilat_payment | Subscription Plans |

---

## kilat_identity — service-identity

### Table: users
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| email | VARCHAR(255) | NO | — | Unique, user email |
| phone | VARCHAR(20) | YES | — | Phone number |
| password_hash | VARCHAR(255) | NO | — | bcrypt hash |
| full_name | VARCHAR(255) | NO | — | Display name |
| role | VARCHAR(20) | NO | — | owner/runner/admin/shop |
| is_verified | BOOLEAN | NO | false | Email verified |
| avatar_url | TEXT | YES | — | Profile picture URL |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

### Table: refresh_tokens
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| user_id | UUID | NO | — | FK to users (indexed) |
| token | TEXT | NO | — | Unique refresh token |
| expires_at | TIMESTAMPTZ | NO | — | Token expiry |
| revoked | BOOLEAN | NO | false | Token revoked |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |

---

## kilat_booking — service-booking

### Table: bookings
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | — | Primary key |
| booking_number | VARCHAR(20) | NO | — | Unique, format BK-XXXXXX |
| owner_id | UUID | NO | — | Pet owner (indexed) |
| runner_id | UUID | YES | — | Assigned runner (indexed) |
| status | VARCHAR(30) | NO | — | State machine (indexed) |
| pet_spec | JSONB | NO | — | {name, petType, weightKg, age, allergies, notes} |
| crate_requirement | JSONB | NO | — | {size, type} |
| pickup_address | JSONB | NO | — | AddressDTO |
| dropoff_address | JSONB | NO | — | AddressDTO |
| route_spec | JSONB | YES | — | {distance, duration, polyline} |
| estimated_price_cents | BIGINT | NO | — | Estimated price |
| final_price_cents | BIGINT | YES | — | Final price |
| currency | VARCHAR(3) | NO | MYR | Currency code |
| scheduled_at | TIMESTAMPTZ | YES | — | Scheduled pickup time |
| picked_up_at | TIMESTAMPTZ | YES | — | When pet was picked up |
| delivered_at | TIMESTAMPTZ | YES | — | When pet was delivered |
| cancelled_at | TIMESTAMPTZ | YES | — | When cancelled |
| cancel_note | VARCHAR(500) | YES | — | Cancellation reason |
| notes | VARCHAR(1000) | YES | — | Additional notes |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

**Status values**: requested, accepted, in_progress, delivered, completed, cancelled

---

## kilat_payment — service-payment

### Table: payments
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| booking_id | UUID | NO | — | Unique, one payment per booking |
| owner_id | UUID | NO | — | Pet owner |
| runner_id | UUID | YES | — | Delivery runner |
| escrow_status | VARCHAR(20) | NO | pending | State machine |
| amount_cents | BIGINT | NO | — | Total amount |
| platform_fee_cents | BIGINT | NO | — | Platform fee |
| runner_payout_cents | BIGINT | NO | — | Runner payout (amount - fee) |
| currency | VARCHAR(3) | NO | MYR | Currency code |
| payment_method | VARCHAR(50) | YES | — | Payment method |
| stripe_payment_id | VARCHAR(255) | YES | — | Mock Stripe payment ID |
| escrow_held_at | TIMESTAMPTZ | YES | — | When escrow was held |
| escrow_released_at | TIMESTAMPTZ | YES | — | When released to runner |
| refunded_at | TIMESTAMPTZ | YES | — | When refunded |
| refund_reason | TEXT | YES | — | Refund reason |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

**Escrow status values**: pending, held, released, refunded, failed

---

## kilat_runner — service-runner

### Table: runners
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| user_id | UUID | NO | — | Unique, FK to identity |
| full_name | VARCHAR(255) | NO | — | Runner name |
| phone | VARCHAR(20) | YES | — | Phone |
| vehicle_type | VARCHAR(20) | NO | — | car/van/motorcycle |
| vehicle_plate | VARCHAR(20) | YES | — | License plate |
| vehicle_model | VARCHAR(100) | YES | — | Vehicle model |
| vehicle_year | INT | YES | — | Vehicle year |
| air_conditioned | BOOLEAN | NO | false | Has AC |
| session_status | VARCHAR(20) | NO | inactive | active/inactive |
| current_lat | DOUBLE | YES | — | GPS latitude |
| current_lng | DOUBLE | YES | — | GPS longitude |
| rating | DECIMAL(3,2) | NO | 0.0 | Average rating |
| total_trips | INT | NO | 0 | Completed trips |
| completion_rate | DECIMAL(3,2) | NO | 1.0 | Trip completion rate |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

**PostGIS**: Uses `ST_DWithin`, `ST_MakePoint`, `ST_Distance` for nearby search

### Table: pet_shops
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| owner_id | UUID | YES | — | Shop owner (indexed) |
| name | VARCHAR(255) | NO | — | Shop name |
| address | TEXT | NO | — | Full address |
| latitude | DOUBLE | NO | — | GPS latitude |
| longitude | DOUBLE | NO | — | GPS longitude |
| phone | VARCHAR(20) | YES | — | Phone |
| email | VARCHAR(255) | YES | — | Email |
| category | VARCHAR(20) | NO | — | grooming/vet/boarding/pet_store (indexed) |
| services | JSONB | NO | [] | Array of service strings |
| rating | DECIMAL(3,2) | NO | 0.0 | Average rating |
| image_url | TEXT | YES | — | Shop image |
| opening_hours | VARCHAR(100) | YES | — | Opening hours |
| description | TEXT | YES | — | Shop description |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

---

## kilat_tracking — service-tracking

### Table: trip_tracks
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| booking_id | UUID | NO | — | Unique, one track per booking |
| runner_id | UUID | NO | — | Runner (indexed) |
| status | VARCHAR(20) | NO | active | active/completed/cancelled (indexed) |
| total_distance_km | DECIMAL(10,3) | NO | 0 | Total distance |
| started_at | TIMESTAMPTZ | NO | NOW() | Trip start |
| completed_at | TIMESTAMPTZ | YES | — | Trip end |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

### Table: waypoints
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| trip_track_id | UUID | NO | — | FK to trip_tracks (indexed) |
| latitude | DOUBLE | NO | — | GPS latitude |
| longitude | DOUBLE | NO | — | GPS longitude |
| speed | DECIMAL(6,2) | YES | — | Speed km/h |
| heading | DECIMAL(5,2) | YES | — | Heading degrees |
| recorded_at | TIMESTAMPTZ | NO | — | GPS timestamp |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |

**PostGIS**: Generates GeoJSON LineString from waypoints

---

## kilat_notification — service-notification

### Table: notifications
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| user_id | UUID | NO | — | Target user (indexed) |
| booking_id | UUID | YES | — | Related booking (indexed) |
| event_type | VARCHAR(50) | NO | — | Source event type |
| title | VARCHAR(255) | NO | — | Notification title |
| body | TEXT | NO | — | Notification body |
| channels_sent | JSONB | NO | [] | Channels successfully sent (push/sms/email) |
| channels_failed | JSONB | NO | [] | Channels that failed |
| retry_count | INT | NO | 0 | Retry attempts |
| max_retries | INT | NO | 3 | Max retries |
| status | VARCHAR(20) | NO | pending | pending/sent/partially_sent/failed |
| is_read | BOOLEAN | NO | false | Read by user |
| metadata | JSONB | YES | — | Extra event data |
| fcm_sent_at | TIMESTAMPTZ | YES | — | Push sent time |
| sms_sent_at | TIMESTAMPTZ | YES | — | SMS sent time |
| email_sent_at | TIMESTAMPTZ | YES | — | Email sent time |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

### Table: notification_preferences
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| user_id | UUID | NO | — | Unique, one per user |
| enable_push | BOOLEAN | NO | true | Enable FCM push |
| enable_sms | BOOLEAN | NO | true | Enable Twilio SMS |
| enable_email | BOOLEAN | NO | true | Enable SMTP email |
| fcm_token | VARCHAR(255) | YES | — | Firebase token |
| phone_number | VARCHAR(20) | YES | — | SMS phone |
| email | VARCHAR(255) | YES | — | Email address |
| quiet_hours_start | TIME | YES | — | Quiet hours start |
| quiet_hours_end | TIME | YES | — | Quiet hours end |
| version | BIGINT | NO | 1 | Optimistic locking |
| created_at | TIMESTAMPTZ | NO | NOW() | Created |
| updated_at | TIMESTAMPTZ | NO | NOW() | Updated |

---

## Architectural Patterns in Database

| Pattern | Usage |
|---------|-------|
| **Optimistic Locking** | All tables have `version` column (BIGINT, default 1) |
| **JSONB Storage** | Booking pet_spec, addresses, route_spec; PetShop services; Notification channels |
| **PostGIS Geospatial** | Runner nearby (ST_DWithin), PetShop nearby, Waypoint GeoJSON |
| **UUID Primary Keys** | All tables use UUID with uuid_generate_v4() |
| **Database per Service** | 6 separate databases, no cross-service joins |
| **Soft State** | Status columns for state machines (booking, payment, tracking) |
