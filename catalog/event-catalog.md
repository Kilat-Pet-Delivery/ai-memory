# Event Catalog - Kilat Pet Delivery
*Complete reference of all Kafka events*

## Event System Overview

**Message Broker**: Apache Kafka 3.x
**Event Format**: CloudEvents specification
**Serialization**: JSON
**Total Events**: 19
**Total Topics**: 4
**Library**: kafka-go

## Event Naming Convention

**Pattern**: `[domain].[action]` — e.g., `booking.requested`, `payment.escrow_held`

## Topics

| # | Topic | Events | Key Producers | Key Consumers |
|---|-------|--------|---------------|---------------|
| 1 | booking.events | 7 | service-booking | notification, tracking, payment |
| 2 | payment.events | 5 | service-payment | booking, notification |
| 3 | runner.events | 4 | service-runner | tracking |
| 4 | tracking.events | 3 | service-tracking | notification |

---

## booking.events (7 events)

### booking.requested
**Producer**: service-booking | **Consumers**: service-notification
**Trigger**: Owner creates a new booking
**Payload**: `booking_id, booking_number, owner_id, pet_type, pet_name, pickup_lat, pickup_lng, dropoff_lat, dropoff_lng, estimated_price_cents, currency, occurred_at`

### booking.accepted
**Producer**: service-booking | **Consumers**: service-notification, service-tracking
**Trigger**: Runner accepts the booking
**Payload**: `booking_id, booking_number, runner_id, owner_id, occurred_at`
**Actions**: Tracking creates TripTrack, Notification alerts owner

### booking.pet_picked_up
**Producer**: service-booking | **Consumers**: service-notification
**Trigger**: Runner picks up the pet
**Payload**: `booking_id, booking_number, runner_id, owner_id, picked_up_at, occurred_at`

### booking.delivery_in_progress
**Producer**: service-booking | **Consumers**: service-notification
**Trigger**: Delivery is in progress (status change)
**Payload**: `booking_id, booking_number, runner_id, owner_id, picked_up_at, occurred_at`

### booking.delivery_confirmed
**Producer**: service-booking | **Consumers**: service-notification, service-payment, service-tracking
**Trigger**: Owner confirms delivery is complete
**Payload**: `booking_id, booking_number, runner_id, owner_id, delivered_at, occurred_at`
**Actions**: Payment releases escrow, Tracking completes trip, Notification alerts both

### booking.completed
**Producer**: service-booking | **Consumers**: service-notification
**Trigger**: Booking fully completed (after escrow released)
**Payload**: `booking_id, booking_number, runner_id, owner_id, final_price_cents, currency, occurred_at`

### booking.cancelled
**Producer**: service-booking | **Consumers**: service-notification, service-payment
**Trigger**: Booking is cancelled by owner or runner
**Payload**: `booking_id, booking_number, cancelled_by, reason, occurred_at`
**Actions**: Payment refunds escrow, Notification alerts parties

---

## payment.events (5 events)

### payment.escrow_created
**Producer**: service-payment | **Consumers**: (none currently)
**Trigger**: Payment record created in DB
**Payload**: `payment_id, booking_id, owner_id, amount_cents, currency, occurred_at`

### payment.escrow_held
**Producer**: service-payment | **Consumers**: service-notification
**Trigger**: Stripe PaymentIntent created, funds held
**Payload**: `payment_id, booking_id, stripe_payment_id, amount_cents, currency, occurred_at`
**Note**: Missing owner_id — notification skipped

### payment.escrow_released
**Producer**: service-payment | **Consumers**: service-booking, service-notification
**Trigger**: Delivery confirmed, funds released to runner
**Payload**: `payment_id, booking_id, runner_id, runner_payout_cents, platform_fee_cents, currency, occurred_at`
**Actions**: Booking marks as completed

### payment.escrow_refunded
**Producer**: service-payment | **Consumers**: service-notification
**Trigger**: Booking cancelled, funds returned to owner
**Payload**: `payment_id, booking_id, owner_id, amount_cents, currency, refund_reason, occurred_at`

### payment.failed
**Producer**: service-payment | **Consumers**: service-notification
**Trigger**: Any saga step fails
**Payload**: `payment_id, booking_id, reason, occurred_at`
**Note**: Missing owner_id — notification skipped

---

## runner.events (4 events)

### runner.online
**Producer**: service-runner | **Consumers**: (none — future use)
**Trigger**: Runner goes online
**Payload**: `runner_id, latitude, longitude, occurred_at`

### runner.offline
**Producer**: service-runner | **Consumers**: (none — future use)
**Trigger**: Runner goes offline
**Payload**: `runner_id, occurred_at`

### runner.location_update
**Producer**: service-runner | **Consumers**: service-tracking
**Trigger**: Runner sends GPS update
**Payload**: `runner_id, booking_id (optional), latitude, longitude, speed_kmh, heading_degrees, timestamp, occurred_at`
**Actions**: Tracking adds waypoint + broadcasts via WebSocket

### runner.assigned
**Producer**: (not yet implemented) | **Consumers**: —
**Payload**: `runner_id, booking_id, occurred_at`

---

## tracking.events (3 events)

### tracking.started
**Producer**: service-tracking | **Consumers**: service-notification
**Trigger**: TripTrack created (when booking accepted)
**Payload**: `track_id, booking_id, runner_id, started_at, occurred_at`
**Note**: Missing owner_id — notification skipped

### tracking.updated
**Producer**: service-tracking | **Consumers**: (intentionally skipped)
**Trigger**: New waypoint added
**Payload**: `track_id, booking_id, runner_id, latitude, longitude, speed_kmh, occurred_at`
**Note**: Too frequent for notifications — handled via WebSocket only

### tracking.completed
**Producer**: service-tracking | **Consumers**: service-notification
**Trigger**: Delivery confirmed, trip completed
**Payload**: `track_id, booking_id, runner_id, total_distance_km, completed_at, occurred_at`
**Note**: Missing owner_id — notification skipped

---

## Event Flow: Complete Booking Lifecycle

```
OWNER creates booking
  → booking.requested ──→ [NOTIFICATION: alerts owner]

RUNNER accepts
  → booking.accepted ──→ [NOTIFICATION: alerts owner]
                     ──→ [TRACKING: creates TripTrack]
                          → tracking.started ──→ [NOTIFICATION]

RUNNER picks up pet
  → booking.pet_picked_up ──→ [NOTIFICATION: alerts owner]

RUNNER sends GPS updates (continuous)
  → runner.location_update ──→ [TRACKING: adds waypoint + WebSocket broadcast]
                                → tracking.updated (WebSocket only, not Kafka notification)

RUNNER marks delivered
  → booking.delivery_in_progress ──→ [NOTIFICATION]

OWNER confirms delivery
  → booking.delivery_confirmed ──→ [NOTIFICATION: alerts both]
                               ──→ [PAYMENT: releases escrow]
                                    → payment.escrow_released ──→ [BOOKING: completes]
                                                               ──→ [NOTIFICATION]
                               ──→ [TRACKING: completes trip]
                                    → tracking.completed ──→ [NOTIFICATION]

BOOKING completes
  → booking.completed ──→ [NOTIFICATION: final summary]
```

## Saga: Payment Escrow

```
HAPPY PATH:
  Owner initiates payment → escrow_created → Stripe hold → escrow_held
  Delivery confirmed → Stripe capture → escrow_released → Booking completes

CANCEL PATH:
  Booking cancelled → Stripe cancel → escrow_refunded

FAILURE PATH:
  Any step fails → payment.failed (compensation applied)
```

## Consumer Groups

| Group ID | Service | Topic | Events Handled |
|----------|---------|-------|----------------|
| service-booking-payments | service-booking | payment.events | escrow_released |
| service-payment-bookings | service-payment | booking.events | delivery_confirmed, cancelled |
| service-tracking-bookings | service-tracking | booking.events | accepted, delivery_confirmed |
| service-tracking-runners | service-tracking | runner.events | location_update |
| service-notification-bookings | service-notification | booking.events | All 7 |
| service-notification-payments | service-notification | payment.events | held, released, refunded, failed |
| service-notification-tracking | service-notification | tracking.events | started, completed |

## Known Gaps

| Issue | Impact | Fix |
|-------|--------|-----|
| Missing owner_id in escrow_held, payment.failed | Notifications skipped | Add owner_id to event payload |
| Missing owner_id in tracking.started/completed | Notifications skipped | Add owner_id to event payload |
| runner.online/offline not consumed | No real-time runner dashboard | Add consumer in tracking service |
| runner.assigned not implemented | Not used | Implement when needed |
| No Dead Letter Queue | Failed messages lost | Add DLQ per topic |
