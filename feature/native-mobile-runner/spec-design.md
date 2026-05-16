# app-runner Native MVP — Design Spec

**Date:** 2026-05-15
**Author:** Luqman (with Claude pair-coding)
**Status:** Draft for review
**Project:** Kilat-Pet-Delivery — native mobile rewrite (portfolio play)

---

## 1. Context

The existing Flutter `app-runner` is a working delivery-partner app for the Kilat Pet Delivery platform. It has 13 screens, talks to 5 backend microservices (identity, runner, booking, tracking, payments/earnings, notifications), uses BLoC + Repository + Clean Architecture, and supports background GPS via Geolocator and realtime tracking via WebSockets.

**This spec covers a native rewrite in Swift (iOS) + Kotlin (Android).** The motivation is portfolio + long-term skill investment, not speed-to-revenue. The Flutter version stays frozen as a reference/spec — it will not be developed further but will not be deleted.

**Goal of MVP:** prove the native architecture works end-to-end on one full happy path:
`login → toggle online → see jobs → accept job → background GPS streaming → realtime WebSocket tracking → mark pickup → mark delivery → see earnings`

Anything outside that path is out of scope for MVP.

---

## 2. Scope decisions

| Decision | Call |
|---|---|
| Platforms | iOS first to MVP completion, then port to Android |
| MVP screen count | 6 (Login, Dashboard, Available Jobs, Job Detail, Active Delivery, Earnings) |
| Auth | JWT + refresh token, matching Flutter |
| Initial accounts | Seeded test runner via direct SQL insert into `service-identity` (user row + bcrypt password hash) and `service-runner` (runner profile row) databases. Seed script documented as the first Phase 0 task. Skip Register / RunnerSetup screens in MVP. |
| Background GPS | `CLLocationManager` (iOS) / `FusedLocationProvider` + Foreground Service (Android) |
| Realtime | `URLSessionWebSocketTask` (iOS) / `OkHttp WebSocket` (Android) |
| Maps | MapKit (iOS) / Google Maps SDK (Android) |
| Local persistence | Out of MVP — only Keychain / EncryptedSharedPreferences for tokens |
| Offline waypoint queue | Out of MVP |
| Push notifications | Out of MVP (Flutter version also lacks FCM) |
| Cancel / reviews / profile edit / notifications screens | Out of MVP |

---

## 3. MVP screen list

1. **Login** — email/phone + password → JWT
2. **Dashboard** — runner online/offline toggle, link to active job if any
3. **Available Jobs** — list of `bookings?status=requested`, refreshable
4. **Job Detail** — booking info + Accept button
5. **Active Delivery** — map with live location + pickup/dropoff pins, Pickup button, Deliver button
6. **Earnings** — paginated list of completed payouts

---

## 4. Architecture — same shape on both platforms

```
View Layer (SwiftUI / Compose)
    ↓
ViewModel Layer (per-screen state + actions, no UI/network imports)
    ↓
Repository Layer (AuthRepo, BookingRepo, RunnerRepo, TrackingRepo, EarningsRepo)
    ↓
Service Layer (HTTP client + WS manager + LocationManager)
    ↓
Storage Layer (Keychain / EncryptedSharedPreferences)
```

Unidirectional data flow. No back-references, no shared singletons except the DI graph (manual on iOS, Hilt on Android).

---

## 5. iOS module structure — `app-runner-ios/KilatRunner/`

```
App/
  KilatRunnerApp.swift          @main, DI composition root
  AppEnvironment.swift          env config (dev/staging/prod base URLs)
Core/
  Network/
    APIClient.swift             URLSession wrapper
    AuthInterceptor.swift       Bearer token + 401 refresh
    Endpoints.swift             typed enum of all endpoints
  WebSocket/
    WebSocketClient.swift       URLSessionWebSocketTask + reconnect
  Location/
    LocationManager.swift       CLLocationManager + background mode
  Storage/
    KeychainStore.swift         access + refresh token persistence
Features/
  Auth/         LoginView, LoginViewModel, AuthRepository
  Dashboard/    DashboardView, DashboardViewModel
  Jobs/         AvailableJobsView, JobDetailView, BookingRepository
  ActiveDelivery/
                ActiveDeliveryView, ActiveDeliveryViewModel, TrackingRepository
  Earnings/     EarningsView, EarningsRepository
Models/         Booking, Runner, TrackingUpdate, etc.
Tests/          ViewModel + Repository unit tests
```

**Stack:** SwiftUI + async/await + `Observable` macro for ViewModels + `URLSession` + `URLSessionWebSocketTask` + `CLLocationManager` + `MapKit` + Keychain.

---

## 6. Android module structure — `app-runner-android/app/src/main/java/my/kilat/runner/`

```
KilatRunnerApp.kt               @HiltAndroidApp
core/
  network/      Retrofit + AuthInterceptor (OkHttp)
  websocket/    OkHttp WebSocket client + reconnect
  location/     FusedLocationProvider + LocationForegroundService
  storage/      EncryptedSharedPreferences token store
features/
  auth/         LoginScreen, LoginViewModel, AuthRepository
  dashboard/    DashboardScreen, DashboardViewModel
  jobs/         AvailableJobsScreen, JobDetailScreen, BookingRepository
  activedelivery/
                ActiveDeliveryScreen, ActiveDeliveryViewModel, TrackingRepository
  earnings/     EarningsScreen, EarningsRepository
models/         Booking, Runner, TrackingUpdate, etc.
di/             Hilt modules (NetworkModule, StorageModule, RepositoryModule)
```

**Stack:** Jetpack Compose + Kotlin Coroutines + Flow + Retrofit + kotlinx.serialization + Hilt + Room (NOT used in MVP, reserved for v2) + Google Maps SDK + FusedLocationProviderClient.

---

## 7. Critical flows

### 7.1 Login

```
LoginView → user enters email/password →
  LoginViewModel.login() →
  AuthRepository.login(email, password) →
  POST /api/v1/auth/login →
  KeychainStore.save(access, refresh) →
  AppEnvironment.session = .authenticated →
  Navigate to Dashboard
```

Error states: invalid credentials → inline error; network offline → retry banner.

### 7.2 Toggle online

```
Dashboard toggle ON →
  Request location permission "Always" (iOS) / "Background" (Android)
  → If granted: RunnerRepository.goOnline() → POST /runners/me/online
  → If denied: show rationale + settings deep-link, toggle stays OFF
```

### 7.3 Accept job + start tracking

```
JobDetailView → tap Accept →
  BookingRepository.accept(bookingId) → POST /bookings/:id/accept →
  Navigate to ActiveDeliveryView →
  ActiveDeliveryViewModel.onAppear:
    1. LocationManager.startUpdates(background: true)
    2. WebSocketClient.connect("/ws/tracking/:bookingId?token=...")
    3. Subscribe to WS TrackingUpdate stream → update map pin
  LocationManager delegate (each waypoint):
    - Buffer in memory (max 5 waypoints OR 30s elapsed)
    - Flush → POST /runners/me/location
```

### 7.4 Mark pickup → mark delivered

```
ActiveDeliveryView "Pickup" button →
  BookingRepository.markPickup(bookingId) → POST /bookings/:id/pickup
  (button changes to "Mark Delivered")

ActiveDeliveryView "Mark Delivered" button →
  BookingRepository.markDelivered(bookingId) → POST /bookings/:id/deliver →
  LocationManager.stopUpdates()
  WebSocketClient.disconnect()
  Navigate back to Dashboard
```

---

## 8. Auth + token refresh

**iOS interceptor pattern:**

```swift
// AuthInterceptor wraps APIClient
// On every request:
//   1. Read access token from Keychain
//   2. Add "Authorization: Bearer <token>" header
//   3. If response is 401:
//      - Lock refresh mutex
//      - POST /api/v1/auth/refresh with refresh token
//      - Save new tokens
//      - Replay original request
//      - If refresh fails (e.g. refresh token expired):
//        - Clear Keychain, post .sessionExpired notification
//        - Force navigation to Login
```

**Android pattern:** OkHttp `Authenticator` for 401 retry + `Interceptor` for header injection. Same logic, different idiom.

Public paths (skip auth header): `/api/v1/auth/login`, `/api/v1/auth/register`, `/api/v1/auth/refresh`.

---

## 9. Background location — platform specifics

### 9.1 iOS

- `Info.plist`:
  - `UIBackgroundModes = ["location"]`
  - `NSLocationAlwaysAndWhenInUseUsageDescription` rationale string
  - `NSLocationWhenInUseUsageDescription` rationale string
- `CLLocationManager`:
  - `desiredAccuracy = kCLLocationAccuracyBest`
  - `distanceFilter = 10` (meters)
  - `allowsBackgroundLocationUpdates = true`
  - `pausesLocationUpdatesAutomatically = false`
  - `showsBackgroundLocationIndicator = true` (blue bar visible while in background)
- Permission flow: when user toggles online, request "Always" permission. iOS may prompt for "When in Use" first then escalate after 2 weeks of actual background usage — accept this OS behavior, don't fight it.

### 9.2 Android

- Manifest permissions:
  - `ACCESS_FINE_LOCATION`
  - `ACCESS_BACKGROUND_LOCATION` (Android 10+)
  - `FOREGROUND_SERVICE`
  - `FOREGROUND_SERVICE_LOCATION` (Android 14+)
  - `POST_NOTIFICATIONS` (Android 13+, for the foreground service notification)
- `LocationForegroundService` extends `Service`:
  - `startForeground()` with persistent notification
  - Subscribes to `FusedLocationProviderClient` updates
  - Posts waypoints to ViewModel via shared flow
- Notification: "Delivery in progress — tap to return" with intent back to ActiveDeliveryScreen

---

## 10. WebSocket realtime

- URL: `ws://{host}/ws/tracking/:bookingId?token={JWT}`
- Connect on `ActiveDeliveryView` appear, disconnect on disappear (or on mark delivered)
- Incoming message format (from `lib-proto/events/tracking`):
  ```json
  {
    "booking_id": "...",
    "runner_id": "...",
    "latitude": 3.139,
    "longitude": 101.687,
    "speed_kmh": 35.2,
    "heading_degrees": 270,
    "timestamp": "2026-05-15T14:23:45Z"
  }
  ```
- Reconnect strategy: exponential backoff (1s, 2s, 4s, 8s, 16s, 30s, then cap), max 5 attempts, then show banner "Reconnecting..." with manual retry
- On app foreground after background: force reconnect

---

## 11. Error handling

| Failure | Behavior |
|---|---|
| Network offline | ViewModel emits `.error(NetworkError.offline)` → View shows retry banner |
| HTTP 401 | Auth interceptor refreshes token, retries once. If still 401 → force logout |
| HTTP 5xx | Show generic error banner with retry |
| WebSocket disconnect | Exponential backoff reconnect (see §10) |
| Location permission denied | Block "go online" toggle, show settings deep-link |
| Background location not granted | Inline rationale + settings deep-link, allow foreground-only mode degraded state |
| App killed mid-delivery | iOS: significant-change location restart. Android: Foreground Service survives unless user force-kills |
| Token refresh returns 401 (refresh expired) | Clear Keychain, navigate to Login |

---

## 12. Testing strategy

- **Unit tests** — every ViewModel + every Repository (with mocked HTTPClient). Target: 70% coverage on Core + Features
- **Integration tests** — 1–2 happy-path tests against staging backend: login flow + booking accept flow
- **UI tests** — out of MVP (slow, brittle early; revisit after architecture stabilizes)
- **Manual smoke checklist** — at `docs/manual-smoke.md`, covers full MVP happy path on a real device

iOS: XCTest. Android: JUnit5 + MockK + Turbine for Flow testing.

---

## 13. Definition of done — MVP ships when:

1. Login works on real iPhone with seeded test account
2. Toggle online → real available jobs appear (from `service-booking`)
3. Accept a job created via Bruno (POST `/bookings` as a test owner account)
4. Active delivery screen: map shows live location, pickup + dropoff pins, route polyline
5. Walking around with phone in pocket and screen off → backend `/runners/me/location` receives waypoints
6. WebSocket connection visible in backend logs; frontend recovers gracefully from forced disconnect
7. Mark pickup → backend booking state advances to `delivery_in_progress`
8. Mark delivered → backend state advances to `awaiting_confirmation`
9. Earnings screen lists the completed booking with payout amount
10. App survives 10-minute background session without crashing or losing GPS

---

## 14. Repo / git layout

- New local folders: `Kilat-Pet-Delivery/app-runner-ios/` and `Kilat-Pet-Delivery/app-runner-android/`
- New GitHub repos under `Kilat-Pet-Delivery` org: `app-runner-ios` and `app-runner-android` (create at first push, not before)
- Existing `app-runner` Flutter repo: untouched, serves as reference. Add a single line to its README: "Frozen as reference for native rewrite — see `app-runner-ios` and `app-runner-android`."
- Push cadence: per phase, not at end (per memory `feedback_per_repo_push_workflow.md`)

---

## 15. Out-of-MVP backlog (don't lose track)

After iOS MVP + Android port:

| Next slice | Notes |
|---|---|
| Register + RunnerSetup flow | New runner onboarding |
| Profile edit | Vehicle, contact details |
| Booking cancel + rebook | Edge-case flows |
| Notifications screen | Notification history + read state |
| APNs (iOS) + FCM (Android) push | New backend integration in service-notification likely needed |
| Offline waypoint queue | Persist queue when offline, flush on reconnect |
| Reviews / ratings | Runner-rates-owner flow |
| Local persistence (SwiftData / Room) | Offline-first feel, cache last job list |
| `app-mobile` native (pet owner) | After runner ships on both platforms |
| `app-shop` native (shop owner) | Lowest priority — internal-ish, Flutter could stay |

---

## 16. Risks + mitigations

| Risk | Mitigation |
|---|---|
| Background GPS gets killed by iOS power manager | Use significant-change location as fallback restart; test on real device |
| Android OEM aggressive battery management (Samsung, Xiaomi) | Document "battery optimization off" requirement, link in Settings; this is a known industry problem |
| WebSocket reconnect storms during network flapping | Cap reconnect attempts; show user-visible state |
| Refresh token rotation race conditions | Lock + queue pending requests during refresh |
| 6–12 month timeline eats into Niaga work | Calendar-bucket explicitly; weekly check-in: "did Kilat eat Niaga time this week?" — if yes, pause Kilat |
| Swift learning curve adds debugging tax | Accept it; expect 2–3x slower first month, then accelerates |
| Apple Developer account approval delay | Sign up early ($99/yr) before you need it for device testing |

---

## 17. Out of scope (explicit)

This spec does NOT cover:
- `app-mobile` (pet owner) native rewrite — separate future spec
- `app-shop` (shop owner) native rewrite — separate future spec
- Web apps (`web-landing`, `web-seller`, `web-runner`, `web-admin`) — stay as Next.js
- Backend changes — backend stays exactly as-is, no API contract changes
- Push notification backend wiring — separate future scope
- App Store / Play Store submission — separate after MVP runs locally

---

## Approval

Approval gate: Luqman reviews this spec before invoking the `writing-plans` skill to produce the phase-by-phase implementation plan.

**Two-plan structure (after approval):**

1. **iOS MVP plan** — `docs/superpowers/plans/2026-05-15-app-runner-ios-mvp-plan.md`
   - Phase 0: dev environment + Xcode project scaffold + DB seed script + GitHub repo
   - Phase 1: Network core (APIClient, AuthInterceptor, Endpoints, KeychainStore)
   - Phase 2: Auth feature (LoginView + ViewModel + AuthRepository)
   - Phase 3: Dashboard + online/offline toggle + RunnerRepository
   - Phase 4: Available Jobs + Job Detail + Accept (BookingRepository)
   - Phase 5: Active Delivery skeleton (map + navigation)
   - Phase 6: Background GPS + waypoint posting (LocationManager + buffer)
   - Phase 7: WebSocket realtime (WebSocketClient + reconnect)
   - Phase 8: Pickup + Deliver actions
   - Phase 9: Earnings screen
   - Phase 10: Manual smoke checklist + polish + first TestFlight build (optional)

2. **Android port plan** — `docs/superpowers/plans/2026-XX-XX-app-runner-android-port-plan.md` (created only after iOS MVP ships)
   - Mirrors the iOS phases but with Kotlin/Compose/Hilt idioms
   - Reuses screen list + flow definitions from this spec
   - Created as a separate plan because: (a) iOS lessons inform Android decisions, (b) keeping plans small and shippable per `feedback_per_repo_push_workflow`, (c) avoids the 3-month-old plan rot problem
