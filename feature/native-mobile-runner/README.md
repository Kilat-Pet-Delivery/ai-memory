# Native Mobile Runner

Native iOS (Swift + SwiftUI) + Android (Kotlin + Compose) rewrite of the Flutter `app-runner` delivery-partner app.

**Motivation:** portfolio / long-term skill investment, not speed-to-revenue. Flutter `app-runner` repo stays frozen as reference; native code lives in new repos: `app-runner-ios` (live) and `app-runner-android` (future).

**Status (2026-05-16):** iOS Phases 0–3 complete (auth + network core + dashboard + 32 tests green). Phase 4 (Available Jobs + Accept) in progress.

## Documents

- [`spec-design.md`](./spec-design.md) — full MVP design spec: scope, architecture, module structure for both platforms, critical flows, error handling, definition of done
- [`plan-ios-mvp.md`](./plan-ios-mvp.md) — iOS-only roadmap: 10 phases × ~40 tasks, file paths + intent + test names + acceptance criteria (no code blocks — code lives in `app-runner-ios` repo)

## Backend contracts the runner app depends on

| Service | Endpoints used |
|---|---|
| service-identity | `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`, `GET /auth/profile` |
| service-runner | `GET /runners/me`, `POST /runners/me/online`, `POST /runners/me/offline`, `POST /runners/me/location` |
| service-booking | `GET /bookings?status=requested`, `GET /bookings/:id`, `POST /bookings/:id/accept`, `POST /bookings/:id/pickup`, `POST /bookings/:id/deliver` |
| service-tracking | `GET /tracking/:bookingId`, WebSocket `/ws/tracking/:bookingId?token=JWT` |
| service-payment | `GET /payments/earnings?page=...&limit=...` |

Backend response envelope: `{ "data": ..., "success": true }` for single objects, `{ "data": [...], "pagination": {...}, "success": true }` for lists.

## Reference

- Flutter `app-runner` source still cited when porting screen UX — it's the canonical spec for "what does each screen look like + which API does it call"
