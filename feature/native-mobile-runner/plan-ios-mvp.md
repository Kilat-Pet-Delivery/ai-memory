# app-runner iOS MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a native iOS app (Swift + SwiftUI) that proves the architecture end-to-end on the runner happy path: login → toggle online → see jobs → accept job → background GPS streaming → realtime WebSocket tracking → mark pickup → mark delivery → see earnings.

**Architecture:** Layered, unidirectional: View (SwiftUI) → ViewModel (@Observable) → Repository → Service (URLSession / URLSessionWebSocketTask / CLLocationManager) → Storage (Keychain). DI via manual constructor injection. iOS 17+ deployment target unlocks @Observable + modern concurrency.

**Tech Stack:** Swift 5.9+, SwiftUI, @Observable macro, async/await, URLSession, URLSessionWebSocketTask, CLLocationManager, MapKit, Keychain (Security framework), XCTest. No third-party deps for MVP — Apple-only stack to learn fundamentals.

**Spec:** `docs/superpowers/specs/2026-05-15-app-runner-native-mvp-design.md`

**Plan format note:** This plan is a roadmap, not a code dump (per feedback memory `feedback_plan_format.md`). Each task lists files, intent, test names, and acceptance criteria. Swift code is written during execution, not pre-written here.

---

## File structure (locked in)

```
app-runner-ios/
  KilatRunner.xcodeproj
  KilatRunner/
    App/
      KilatRunnerApp.swift       @main entry
      AppEnvironment.swift       base URL config (debug/release)
      AppSession.swift           @Observable auth state holder
      RootView.swift             routes login vs authenticated
    Core/
      Network/
        HTTPMethod.swift         enum
        APIEndpoint.swift        enum of all MVP endpoints + method/path/auth
        NetworkError.swift       error types with user-facing messages
        APIClient.swift          URLSession wrapper, generic request<T>
        AuthInterceptor.swift    Bearer injection + 401 refresh-and-retry
      WebSocket/
        WebSocketClient.swift    URLSessionWebSocketTask wrapper + reconnect
      Location/
        LocationManager.swift    CLLocationManager wrapper, background mode
        WaypointBuffer.swift     buffers waypoints, flushes by count or time
      Storage/
        KeychainStore.swift      access + refresh token persistence
    Features/
      Auth/                      AuthModels, AuthRepository, LoginViewModel, LoginView
      Dashboard/                 RunnerModels, RunnerRepository, DashboardViewModel, DashboardView
      Jobs/                      BookingModels, BookingRepository, AvailableJobs(VM+View), JobDetail(VM+View)
      ActiveDelivery/            TrackingModels, TrackingRepository, ActiveDeliveryViewModel, ActiveDeliveryView
      Earnings/                  EarningsModels, EarningsRepository, EarningsViewModel, EarningsView
    Resources/
      Info.plist
      Assets.xcassets
  KilatRunnerTests/
    Core/                        Keychain, APIClient, AuthInterceptor, WaypointBuffer, WebSocket
    Features/                    one VM test file + one Repository test file per feature
  docs/
    manual-smoke.md
  README.md
```

**Decomposition principle:** one file = one responsibility. ViewModels never import URLSession; Repositories never import SwiftUI; Views never import Network.

---

## Conventions

- **TDD where it pays:** ViewModels, Repositories, Network code, Buffers, Storage → unit tests first. Views → manual smoke only (UI tests skipped for MVP).
- **Mocking:** URLProtocol stub for network, in-memory KeychainStore variant for tests, protocol-based abstraction for CLLocationManager + URLSessionWebSocketTask.
- **Commits:** small, frequent, conventional-commits style (`feat:`, `fix:`, `chore:`, `test:`). Commit at the end of each task, push at the end of each phase (per `feedback_per_repo_push_workflow.md`).
- **DI:** manual constructor injection. No DI framework. `AppSession` is the only shared @Observable, passed via `.environment(...)`.
- **Date encoding:** ISO8601 both directions; snake_case ↔ camelCase via JSONDecoder/Encoder strategies.

---

## Phase 0 — Project bootstrapping

**Goal:** dev environment proven, backend running locally, test runner seeded, empty Xcode project + GitHub repo live.

### Task 0.1: Pre-flight checks (no files)
- Verify Xcode 15+ installed and selected (`xcode-select -p`).
- Verify iOS Simulator works (`xcrun simctl list devices available`).
- Start Apple Developer Program enrollment ($99/yr) — 24–48h approval, kick off now so it's ready for Phase 6 device testing.
- Confirm `infrastructure/Makefile` has `up` / `down` / `seed` targets.

**Done when:** all 3 verification commands return expected output; Apple Dev enrollment submitted.

### Task 0.2: Bring up backend stack
**Files:** none (uses existing infrastructure).
- Run `make up` in `Kilat-Pet-Delivery/infrastructure/`. Wait for compose to start Postgres + Kafka + 7 services + api-gateway.
- Verify gateway health: `curl http://localhost:8080/health` returns all services healthy.
- If any service unhealthy, capture in `docs/dev-setup-notes.md` and resolve.

**Done when:** all upstream services report healthy via gateway.

### Task 0.3: Seed test runner account
**Files:**
- Create `infrastructure/seed/runner-test-user.sql` — inserts user row into service-identity DB + runner row into service-runner DB. Email `runner.test@kilat.my`, password `TestRunner123!` (bcrypt hashed).

**Intent:** Skip Register/RunnerSetup screens in MVP — log in as a pre-seeded runner.

**Done when:** `curl POST /api/v1/auth/login` with seed credentials returns access+refresh tokens.

### Task 0.4: Create Xcode project
**Files (created via Xcode wizard):**
- `app-runner-ios/KilatRunner.xcodeproj`
- `app-runner-ios/KilatRunner/` (source folder)
- `app-runner-ios/KilatRunnerTests/`

**Settings:** Bundle ID `my.kilat.KilatRunner`, iOS 17+ deployment target, SwiftUI interface, Storage=None, Include Tests=YES. Add Background Modes → Location updates capability now (used in Phase 6).

**Done when:** default "Hello, world!" SwiftUI app launches on iPhone 15 Simulator.

### Task 0.5: Initial git + GitHub repo
**Files:**
- Create `.gitignore` — Xcode + DerivedData + xcuserdata + .DS_Store + .env.
- Create `README.md` — project description, build instructions, test runner credentials, links to spec + plan.

**Steps:** `git init -b main`, add scaffold, commit, then `gh repo create Kilat-Pet-Delivery/app-runner-ios --public --source=. --remote=origin --push`.

**Done when:** repo visible at github.com/Kilat-Pet-Delivery/app-runner-ios with scaffold pushed.

**Phase 0 push checkpoint** ✅

---

## Phase 1 — Network core

**Goal:** typed HTTP client with auth interceptor + Keychain-backed token storage, all unit-tested. Zero feature code yet.

### Task 1.1: AppEnvironment
**Files:** `KilatRunner/App/AppEnvironment.swift`
**Intent:** Static config holder. `baseURL` = `http://localhost:8080` (DEBUG) / `https://api.kilat.my` (RELEASE). Derive `apiBaseURL` (+`/api/v1`) and `wsBaseURL` (swap http→ws).
**Tests:** none — pure static config.
**Done when:** compiles, `AppEnvironment.apiBaseURL` resolves correctly in debug + release builds.

### Task 1.2: HTTPMethod + APIEndpoint
**Files:** `KilatRunner/Core/Network/HTTPMethod.swift`, `KilatRunner/Core/Network/APIEndpoint.swift`
**Intent:** Enum-typed endpoint catalog. `APIEndpoint` cases cover all MVP endpoints: `login`, `refresh`, `logout`, `profile`, `runnerMe`, `runnerOnline`, `runnerOffline`, `runnerLocation`, `availableJobs`, `bookingDetail(id)`, `acceptBooking(id)`, `markPickup(id)`, `markDelivered(id)`, `earnings(page, limit)`. Each case exposes `path`, `method`, `queryItems`, `requiresAuth`.
**Tests:** none — pure mapping. Visual review of path strings against backend Bruno/curl.
**Done when:** compiles, all 14 endpoints listed, login + refresh marked `requiresAuth = false`.

### Task 1.3: NetworkError
**Files:** `KilatRunner/Core/Network/NetworkError.swift`
**Intent:** Error enum with cases `offline`, `invalidURL`, `invalidResponse`, `unauthorized`, `forbidden`, `notFound`, `serverError(Int)`, `decodingFailed(String)`, `encodingFailed(String)`, `unknown(String)`. Each has a `userMessage` computed property for view layer.
**Tests:** none — pure enum.
**Done when:** Equatable conformance compiles; every case has a non-empty user message.

### Task 1.4: KeychainStore (TDD)
**Files:**
- `KilatRunner/Core/Storage/KeychainStore.swift` — wraps SecItem APIs with `saveAccessToken`, `accessToken`, `saveRefreshToken`, `refreshToken`, `clear`. Service identifier configurable for test isolation.
- `KilatRunnerTests/Core/KeychainStoreTests.swift`

**Test names + assertions:**
- `test_saveAndReadAccessToken_returnsSameValue` — write then read returns same string.
- `test_saveAndReadRefreshToken_returnsSameValue` — same for refresh slot.
- `test_overwriteToken_returnsNewValue` — second write overwrites first.
- `test_clear_removesBothTokens` — both reads return nil after clear.
- `test_readUnsetToken_returnsNil` — empty keychain returns nil.

**Done when:** all 5 tests green, `kSecAttrAccessibleAfterFirstUnlock` used for accessibility.

### Task 1.5: MockURLProtocol test helper
**Files:** `KilatRunnerTests/Core/MockURLProtocol.swift`
**Intent:** URLProtocol subclass that captures requests and returns canned (HTTPURLResponse, Data?) from a static handler closure. Used by APIClient + AuthInterceptor tests.
**Tests:** none directly — exercised through APIClient tests.
**Done when:** compiles; `MockURLProtocol.requestHandler` + `MockURLProtocol.capturedRequests` + `MockURLProtocol.reset()` exposed.

### Task 1.6: APIClient (TDD)
**Files:**
- `KilatRunner/Core/Network/APIClient.swift` — generic `request<Body: Encodable, Response: Decodable>(_ endpoint:, body:, token:) async throws -> Response`. URL building, Bearer injection when token provided + `requiresAuth`, status-code mapping to NetworkError, JSON encode/decode with snake_case + ISO8601.
- `KilatRunnerTests/Core/APIClientTests.swift`

**Test names + assertions:**
- `test_get_success_decodesPayload` — 200 + JSON body → decoded payload returned.
- `test_get_returnsUnauthorized_on401` — 401 status throws `NetworkError.unauthorized`.
- `test_get_returnsServerError_on500` — 500 status throws `NetworkError.serverError(500)`.
- `test_request_addsBearerToken_whenProvided` — `Authorization: Bearer <token>` header present in captured request.
- `test_request_omitsBearer_whenNoToken` — no Authorization header when token is nil.
- `test_request_encodesBodySnakeCase` — `camelCaseField` becomes `camel_case_field` in JSON body.
- `test_request_decodesSnakeCaseResponse` — `access_token` JSON field decodes to `accessToken` Swift property.

**Done when:** all 7 tests green; APIClient takes a `URLSession` constructor param so tests can inject `MockURLProtocol`.

### Task 1.7: AuthInterceptor (TDD)
**Files:**
- `KilatRunner/Core/Network/AuthInterceptor.swift` — wraps APIClient. On 401 for auth-required endpoint, atomically refreshes via `/auth/refresh`, updates keychain, retries original request once. On refresh failure, clears keychain and throws `.unauthorized`. Refresh lock (NSLock) to serialize concurrent refresh attempts.
- `KilatRunnerTests/Core/AuthInterceptorTests.swift`

**Test names + assertions:**
- `test_perform_addsCurrentTokenAndSucceeds` — happy path with valid token.
- `test_perform_on401_refreshesTokenAndRetries` — 401 → refresh call observed → original request retried with new token → success. Keychain holds new tokens after.
- `test_perform_refreshFails_throwsUnauthorizedAndClears` — refresh returns 401 → keychain cleared → `.unauthorized` thrown.
- `test_perform_concurrentRequests_refreshOnce` — two concurrent 401s trigger ONE refresh call (not two).

**Done when:** all 4 tests green; concurrent-refresh test exercises the NSLock.

### Task 1.8: AppSession + RootView
**Files:**
- `KilatRunner/App/AppSession.swift` — `@Observable` class with `state: .unauthenticated | .authenticated`, initialized from keychain presence. Methods: `markAuthenticated()`, `logout()` (clears keychain + flips state).
- `KilatRunner/App/RootView.swift` — switches on `session.state`, shows placeholder text for now ("Login screen placeholder" / "Authenticated home placeholder").
- Modify `KilatRunner/KilatRunnerApp.swift` — wire `@State private var session = AppSession()` into `.environment(session)`.

**Tests:** none — trivial state holder, covered indirectly via auth feature tests in Phase 2.

**Done when:** app launches to "Login screen placeholder" on fresh install; injecting a token into keychain (via debugger) and relaunching shows "Authenticated home placeholder".

**Phase 1 push checkpoint** ✅ — 16+ tests green, network core complete, zero feature code yet.

---

## Phase 2 — Auth feature

**Goal:** real login screen → POST `/auth/login` → tokens in keychain → `AppSession` flips to authenticated → RootView shows authenticated tree.

### Task 2.1: Auth models + repository (TDD)
**Files:**
- `KilatRunner/Features/Auth/AuthModels.swift` — `LoginRequest { email, password }`, `LoginResponse { accessToken, refreshToken, user }`, `AuthenticatedUser { id, email, fullName, role }`.
- `KilatRunner/Features/Auth/AuthRepository.swift` — `func login(email:password:) async throws -> AuthenticatedUser`. Internally: call APIClient via AuthInterceptor `.login`, save tokens to keychain on success.
- `KilatRunnerTests/Features/AuthRepositoryTests.swift`

**Test names + assertions:**
- `test_login_success_savesTokensToKeychain` — 200 + token JSON → both access+refresh saved to keychain → returned user matches response.
- `test_login_wrongPassword_throwsUnauthorized_keychainUntouched` — 401 → keychain remains empty → `.unauthorized` thrown.
- `test_login_serverError_throwsServerError_keychainUntouched` — 500 → keychain remains empty.

**Done when:** all 3 tests green.

### Task 2.2: LoginViewModel (TDD)
**Files:**
- `KilatRunner/Features/Auth/LoginViewModel.swift` — `@Observable` with `email`, `password`, `isSubmitting`, `errorMessage`. `login()` async method calls AuthRepository, on success calls `appSession.markAuthenticated()`, on error sets `errorMessage`.
- `KilatRunnerTests/Features/LoginViewModelTests.swift`

**Test names + assertions:**
- `test_login_success_setsSessionAuthenticated` — repository returns success → `appSession.state == .authenticated` → `isSubmitting == false`.
- `test_login_failure_setsErrorMessage_doesNotAuthenticate` — repository throws → `errorMessage` populated with user-facing message → session unchanged.
- `test_login_setsIsSubmittingDuringCall` — `isSubmitting` is true mid-call, false after.
- `test_login_emptyEmailOrPassword_setsValidationError_skipsNetwork` — validation guard before any repository call.

**Mocking strategy:** AuthRepository takes a protocol/closure so test passes a mock that returns canned values.

**Done when:** all 4 tests green.

### Task 2.3: LoginView (SwiftUI)
**Files:** `KilatRunner/Features/Auth/LoginView.swift`
**Intent:** Email TextField + SecureField + login button + inline error text + activity indicator while submitting. Bind to LoginViewModel via `@Bindable`.
**Tests:** none — manual smoke.
**Done when:** Renders correctly in Xcode Preview; tapping login with seed credentials advances to authenticated placeholder (visible to manual test).

### Task 2.4: Wire RootView to real screens
**Files:** modify `KilatRunner/App/RootView.swift`
**Intent:** Replace "Login screen placeholder" with `LoginView`. Leave authenticated branch as a temporary placeholder until Phase 3.
**Tests:** none.
**Done when:** Manual smoke: launch app → log in with seed account → reach placeholder authenticated screen → kill app + relaunch → still authenticated (token persisted).

**Phase 2 push checkpoint** ✅ — login works end-to-end against local backend.

---

## Phase 3 — Dashboard + online/offline toggle

**Goal:** authenticated home screen with online/offline switch wired to `/runners/me/online` and `/runners/me/offline`, location permission requested when toggling on.

### Task 3.1: Runner models + repository (TDD)
**Files:**
- `KilatRunner/Features/Dashboard/RunnerModels.swift` — `Runner { id, userId, vehicleType, maxCrateCapacity, isOnline, currentLatitude, currentLongitude }`, `OnlineStatus enum { online, offline }`.
- `KilatRunner/Features/Dashboard/RunnerRepository.swift` — `getMe() async throws -> Runner`, `goOnline() async throws`, `goOffline() async throws`, `postLocation(_ waypoint:) async throws`.
- `KilatRunnerTests/Features/RunnerRepositoryTests.swift`

**Test names + assertions:**
- `test_getMe_returnsRunner` — 200 + runner JSON → decoded runner returned.
- `test_goOnline_postsToCorrectEndpoint` — captured request path equals `/runners/me/online`, method POST.
- `test_goOffline_postsToCorrectEndpoint` — captured request path equals `/runners/me/offline`.
- `test_postLocation_sendsLatLngHeadingSpeed` — request body JSON contains all 4 fields snake-cased.

**Done when:** all 4 tests green.

### Task 3.2: DashboardViewModel (TDD)
**Files:**
- `KilatRunner/Features/Dashboard/DashboardViewModel.swift` — `@Observable` with `runner`, `isOnline`, `errorMessage`, `isTogglingOnline`. Methods: `loadRunner()`, `toggleOnline()` (calls go-online or go-offline based on current state; on go-online, requests location "always" permission BEFORE the API call — fail closed if denied).
- `KilatRunnerTests/Features/DashboardViewModelTests.swift`

**Test names + assertions:**
- `test_loadRunner_populatesRunnerState` — repository returns runner → VM state updated.
- `test_toggleOnline_whenOffline_requestsPermissionThenCallsGoOnline` — permission granted → goOnline called once → isOnline = true.
- `test_toggleOnline_permissionDenied_doesNotCallGoOnline_setsError` — permission denied → goOnline never called → errorMessage set.
- `test_toggleOnline_whenOnline_callsGoOffline_noPermissionCheck` — already-online toggle just calls goOffline.

**Mocking strategy:** inject `LocationPermissionProvider` protocol with `requestAlwaysAuthorization() async -> Bool`.

**Done when:** all 4 tests green.

### Task 3.3: DashboardView + permission rationale
**Files:**
- `KilatRunner/Features/Dashboard/DashboardView.swift` — top bar with runner name + role, big online/offline toggle, secondary buttons "View Available Jobs" + "Earnings". Settings deep-link banner when permission denied.
- `KilatRunner/Features/Dashboard/PermissionRationaleSheet.swift` — modal explaining why "Always" location is needed before iOS prompt fires.
- Modify `KilatRunner/App/RootView.swift` — authenticated branch now shows `DashboardView` (via NavigationStack).

**Tests:** none — manual smoke.
**Done when:** Manual smoke: log in → see dashboard → toggle online → iOS prompts for location → grant → toggle reflects online state → kill + relaunch → still authenticated, dashboard loads runner.

**Phase 3 push checkpoint** ✅

---

## Phase 4 — Available Jobs + Accept

**Goal:** list available bookings, view detail, accept a booking → navigate to placeholder Active Delivery screen.

### Task 4.1: Booking models + repository (TDD)
**Files:**
- `KilatRunner/Features/Jobs/BookingModels.swift` — `Booking { id, status, petType, petName, pickupAddress, pickupLatitude, pickupLongitude, dropoffAddress, dropoffLatitude, dropoffLongitude, fareCents, distanceMeters, createdAt }`. `BookingStatus enum { requested, accepted, deliveryInProgress, awaitingConfirmation, completed, cancelled }`.
- `KilatRunner/Features/Jobs/BookingRepository.swift` — `listAvailable() async throws -> [Booking]`, `get(id:) async throws -> Booking`, `accept(id:) async throws -> Booking`, `markPickup(id:) async throws -> Booking`, `markDelivered(id:) async throws -> Booking`.
- `KilatRunnerTests/Features/BookingRepositoryTests.swift`

**Test names + assertions:**
- `test_listAvailable_decodesArray` — 200 + JSON array → list of Booking returned.
- `test_get_decodesSingleBooking` — `/bookings/:id` decoded.
- `test_accept_postsToAcceptEndpoint_returnsUpdatedBooking` — captured path equals `/bookings/<id>/accept`, returned booking has status `.accepted`.
- `test_markPickup_postsToPickupEndpoint` — captured path equals `/bookings/<id>/pickup`.
- `test_markDelivered_postsToDeliverEndpoint` — captured path equals `/bookings/<id>/deliver`.
- `test_status_decodesAllCases` — JSON strings decode to all BookingStatus enum cases.

**Done when:** all 6 tests green.

### Task 4.2: AvailableJobsViewModel + AvailableJobsView
**Files:**
- `KilatRunner/Features/Jobs/AvailableJobsViewModel.swift` — `@Observable` with `jobs`, `isLoading`, `errorMessage`. Methods: `load()`, `refresh()`.
- `KilatRunner/Features/Jobs/AvailableJobsView.swift` — pull-to-refresh List, each row shows pickup→dropoff + fare + distance. Tap row → push JobDetailView.
- `KilatRunnerTests/Features/AvailableJobsViewModelTests.swift`

**Test names + assertions:**
- `test_load_populatesJobs` — repository returns array → VM jobs == array → isLoading false after.
- `test_load_emptyList_isHandled` — empty array → no error, jobs == [].
- `test_load_failure_setsError` — repository throws → errorMessage populated, jobs stays empty.

**Done when:** all 3 tests green + manual smoke: dashboard → "View Available Jobs" → see list (will be empty until a booking is created via Bruno/curl).

### Task 4.3: JobDetailViewModel + JobDetailView
**Files:**
- `KilatRunner/Features/Jobs/JobDetailViewModel.swift` — `@Observable` with `booking`, `isAccepting`, `errorMessage`, `acceptedBookingId` (signal for navigation).
- `KilatRunner/Features/Jobs/JobDetailView.swift` — booking summary card + pickup/dropoff map preview + big Accept button. After accept, navigate to ActiveDeliveryView with the accepted booking.

**Test names + assertions:**
- `test_accept_success_setsAcceptedBookingId` — repository accept returns booking → `acceptedBookingId` set → isAccepting false.
- `test_accept_failure_setsError_doesNotSetAcceptedId` — error → errorMessage populated, no navigation signal.
- `test_accept_setsIsAcceptingDuringCall` — flag flips correctly.

**Done when:** all 3 tests green + manual smoke: create a booking via Bruno → see it in available jobs → tap → see detail → Accept → land on ActiveDelivery placeholder.

### Task 4.4: Wire navigation
**Files:** modify `DashboardView` and `RootView`/`AvailableJobsView` for NavigationStack.
**Intent:** NavigationStack-based routing: Dashboard → AvailableJobs → JobDetail → ActiveDelivery (placeholder for now).
**Done when:** Full flow navigable on simulator.

**Phase 4 push checkpoint** ✅

---

## Phase 5 — Active Delivery skeleton

**Goal:** Active Delivery screen with MapKit map showing pickup + dropoff pins, placeholder "live location" marker, Pickup + Mark Delivered buttons (not yet functional — wired in Phase 8).

### Task 5.1: Tracking models + repository skeleton
**Files:**
- `KilatRunner/Features/ActiveDelivery/TrackingModels.swift` — `TrackingUpdate { bookingId, runnerId, latitude, longitude, speedKmh, headingDegrees, timestamp }`.
- `KilatRunner/Features/ActiveDelivery/TrackingRepository.swift` — placeholder: `getHistory(bookingId:) async throws -> [TrackingUpdate]` (used later; not in MVP critical path but cheap to add now).
- `KilatRunnerTests/Features/TrackingRepositoryTests.swift`

**Test names + assertions:**
- `test_getHistory_decodesArray` — 200 + JSON → tracking updates returned.
- `test_trackingUpdate_decodesFromSnakeCaseJSON` — `speed_kmh` and `heading_degrees` decode to camelCase Swift props.

**Done when:** both tests green.

### Task 5.2: ActiveDeliveryViewModel skeleton
**Files:**
- `KilatRunner/Features/ActiveDelivery/ActiveDeliveryViewModel.swift` — `@Observable` with `booking`, `currentLocation` (CLLocationCoordinate2D?), `pickupCoordinate`, `dropoffCoordinate`, `deliveryPhase enum { enroute, pickedUp, delivered }`, `errorMessage`.
- `KilatRunnerTests/Features/ActiveDeliveryViewModelTests.swift`

**Test names + assertions (Phase 5 baseline):**
- `test_init_derivesCoordinatesFromBooking` — coordinates extracted from Booking match constructor input.
- `test_initialDeliveryPhase_isEnroute` — fresh VM starts in `.enroute`.

**Done when:** both tests green. (Phase 6/7/8 will add more tests to this same file.)

### Task 5.3: ActiveDeliveryView with MapKit
**Files:** `KilatRunner/Features/ActiveDelivery/ActiveDeliveryView.swift`
**Intent:** SwiftUI `Map` view with `Marker` for pickup + dropoff. Floating cards at top (pet info, pickup/dropoff addresses) and bottom (Pickup button → Mark Delivered button, state-driven). Buttons present but non-functional (will wire in Phase 8).
**Tests:** none — manual smoke.
**Done when:** Manual smoke: accept a booking via Phase 4 flow → land on Active Delivery → see two pins on map + booking info card.

**Phase 5 push checkpoint** ✅

---

## Phase 6 — Background GPS

**Goal:** real-device-tested background location streaming. When delivery active, app posts batched waypoints to `/runners/me/location` even with screen off / app backgrounded.

### Task 6.1: Info.plist + capability config
**Files:** modify `KilatRunner/Resources/Info.plist`
**Intent:**
- Add `UIBackgroundModes = ["location"]` (should already be set from Phase 0 Task 0.4 capability).
- Add `NSLocationWhenInUseUsageDescription` — short rationale ("Track your current location while delivering").
- Add `NSLocationAlwaysAndWhenInUseUsageDescription` — longer rationale ("KilatRunner tracks your location in the background to update customers on delivery status. Required while you're on an active delivery.").

**Done when:** Privacy strings visible in Info.plist; build succeeds with no warnings.

### Task 6.2: LocationManager wrapper (TDD via protocol)
**Files:**
- `KilatRunner/Core/Location/LocationManager.swift` — wraps `CLLocationManager`. Exposes a `locationUpdates: AsyncStream<CLLocation>`. Methods: `requestAlwaysAuthorization() async -> CLAuthorizationStatus`, `startUpdates()`, `stopUpdates()`. Sets `allowsBackgroundLocationUpdates = true`, `desiredAccuracy = kCLLocationAccuracyBest`, `distanceFilter = 10`, `pausesLocationUpdatesAutomatically = false`, `showsBackgroundLocationIndicator = true`.
- Define a `LocationProvider` protocol that LocationManager conforms to — VM tests inject a fake.

**Tests:** unit tests only on the protocol-conforming fake (real CLLocationManager hard to test).
**Done when:** Manual smoke: tap online toggle in Dashboard → see iOS prompt for "When in Use" → upgrade to "Always" via Settings → status reflected in app.

### Task 6.3: WaypointBuffer (TDD)
**Files:**
- `KilatRunner/Core/Location/WaypointBuffer.swift` — collects waypoints; flushes when count ≥ 5 OR elapsed time ≥ 30s, whichever first. Flush callback async — caller (ViewModel) does the POST. Thread-safe via actor.
- `KilatRunnerTests/Core/WaypointBufferTests.swift`

**Test names + assertions:**
- `test_addLessThan5Waypoints_doesNotFlush` — 4 adds → callback not invoked.
- `test_addExactly5Waypoints_flushesOnce` — 5th add triggers callback with all 5.
- `test_after30Seconds_flushesEvenIfUnderCount` — using injected clock, advance 30s after 2 adds → callback fires with 2.
- `test_flush_clearsBuffer` — after flush, next add starts a fresh batch.
- `test_concurrentAdds_thread_safe` — 10 concurrent adds from tasks → exactly 10 waypoints batched (no duplication, no loss).

**Done when:** all 5 tests green.

### Task 6.4: Wire LocationManager + WaypointBuffer into ActiveDeliveryViewModel
**Files:** modify `KilatRunner/Features/ActiveDelivery/ActiveDeliveryViewModel.swift`
**Intent:** On view appear, start LocationManager updates. Each waypoint → update `currentLocation` (for map pin) → push into WaypointBuffer. On flush, call `runnerRepository.postLocation(_:)`. On view disappear (or `.delivered` phase), stop updates + final flush.
**Test additions:**
- `test_onAppear_startsLocationUpdates` — fake LocationProvider's `startUpdates()` called.
- `test_locationUpdate_updatesCurrentLocationAndBuffersWaypoint` — emitting a location via fake → `currentLocation` updated + buffer received it.
- `test_onDelivered_stopsUpdatesAndFlushes` — phase transitions to `.delivered` → stopUpdates called → final flush invoked.

**Done when:** all 3 new tests green + real-device smoke: walk a block with app backgrounded → backend `/runners/me/location` logs show waypoints arriving.

**Phase 6 push checkpoint** ✅ — requires Apple Developer account for device testing.

---

## Phase 7 — WebSocket realtime

**Goal:** Active Delivery screen shows live position via WebSocket subscription (in addition to its own GPS). Reconnect with backoff on drops.

### Task 7.1: WebSocketClient (TDD via protocol abstraction)
**Files:**
- `KilatRunner/Core/WebSocket/WebSocketClient.swift` — wraps `URLSessionWebSocketTask`. Exposes `connect(url: URL) async throws`, `disconnect()`, `messages: AsyncStream<Data>`. Auto-reconnects with exponential backoff (1s, 2s, 4s, 8s, 16s, cap 30s; max 5 attempts before surfacing as user-visible disconnect).
- Define `WebSocketTransport` protocol; production impl uses URLSessionWebSocketTask, test impl is in-memory.
- `KilatRunnerTests/Core/WebSocketClientTests.swift`

**Test names + assertions:**
- `test_connect_emitsReceivedMessages` — fake transport delivers a message → AsyncStream yields it.
- `test_disconnect_stopsEmittingMessages` — after disconnect, no further yields.
- `test_drop_triggersReconnectWithBackoff` — fake transport drops; injected clock advances; reconnect attempted at 1s, 2s, 4s.
- `test_exceedMaxAttempts_surfacesDisconnectedState` — 5 failed reconnects → state observable becomes `.disconnected`, no more retries.

**Done when:** all 4 tests green.

### Task 7.2: TrackingUpdate decoder
**Files:** reuse `KilatRunner/Features/ActiveDelivery/TrackingModels.swift`. Add a small decoder helper (or just JSONDecoder().decode at the call site).
**Tests:** covered by Phase 5 Task 5.1 `test_trackingUpdate_decodesFromSnakeCaseJSON`.
**Done when:** no new file; verify the decoder handles the actual WebSocket payload (capture one via Bruno or backend logs first).

### Task 7.3: Wire WebSocket into ActiveDeliveryViewModel
**Files:** modify `KilatRunner/Features/ActiveDelivery/ActiveDeliveryViewModel.swift`
**Intent:** On view appear, also `WebSocketClient.connect(wsBaseURL + "/ws/tracking/<bookingId>?token=<jwt>")`. Each decoded TrackingUpdate updates `currentLocation` (same property the GPS updates — they cooperate). On disappear, `disconnect()`.
**Test additions:**
- `test_onAppear_connectsWebSocket` — fake WS transport sees connect with correct URL + token query param.
- `test_incomingTrackingUpdate_updatesCurrentLocation` — emit a TrackingUpdate JSON → `currentLocation` reflects it.
- `test_onDisappear_disconnectsWebSocket` — disconnect called.

**Done when:** all 3 new tests green + manual smoke: with delivery active on simulator and backend running, force a tracking event via backend cli/Bruno → see map marker move.

**Phase 7 push checkpoint** ✅

---

## Phase 8 — Pickup + Mark Delivered

**Goal:** Pickup button → `/bookings/:id/pickup` → state advances. Mark Delivered → `/bookings/:id/deliver` → state advances → cleanup (stop GPS, disconnect WS) → navigate back to Dashboard.

### Task 8.1: Pickup action
**Files:** modify `KilatRunner/Features/ActiveDelivery/ActiveDeliveryViewModel.swift`
**Intent:** `markPickup() async` method. Calls `BookingRepository.markPickup(id:)`. On success: `deliveryPhase = .pickedUp`. On failure: errorMessage.
**Test additions:**
- `test_markPickup_success_advancesPhase` — repo returns booking with status .deliveryInProgress → phase becomes `.pickedUp`.
- `test_markPickup_failure_keepsPhase_setsError` — error → phase unchanged, errorMessage populated.

**Done when:** both tests green; ActiveDeliveryView's Pickup button calls VM method.

### Task 8.2: Mark Delivered action + cleanup
**Files:** modify `KilatRunner/Features/ActiveDelivery/ActiveDeliveryViewModel.swift`
**Intent:** `markDelivered() async`. Calls `BookingRepository.markDelivered(id:)`. On success: `deliveryPhase = .delivered`, stop LocationManager, disconnect WebSocketClient, flush waypoint buffer one last time. On failure: errorMessage.
**Test additions:**
- `test_markDelivered_success_stopsLocationAndDisconnectsWS` — fakes for LocationProvider + WSTransport see stop + disconnect called.
- `test_markDelivered_success_finalFlushOfBuffer` — buffer flush called even if under 5 waypoints / under 30s.
- `test_markDelivered_failure_keepsResourcesActive` — error → location still running, WS still connected.

**Done when:** all 3 tests green.

### Task 8.3: Navigate back to Dashboard on delivered
**Files:** modify `KilatRunner/Features/ActiveDelivery/ActiveDeliveryView.swift`
**Intent:** When `deliveryPhase == .delivered`, show a "Trip complete" overlay with payout summary + "Back to Dashboard" button. Tap → `NavigationStack` pops to root.
**Tests:** none — manual smoke.
**Done when:** Full manual smoke (the §13 Definition of Done from the spec, steps 1–8) passes end-to-end on a real device with backend running.

**Phase 8 push checkpoint** ✅

---

## Phase 9 — Earnings

**Goal:** Earnings screen lists completed payouts with pagination.

### Task 9.1: Earnings model + repository (TDD)
**Files:**
- `KilatRunner/Features/Earnings/EarningsModels.swift` — `Earning { id, bookingId, amountCents, currency, status, completedAt }`. `EarningsPage { items, page, limit, total }`.
- `KilatRunner/Features/Earnings/EarningsRepository.swift` — `list(page:limit:) async throws -> EarningsPage`.
- `KilatRunnerTests/Features/EarningsRepositoryTests.swift`

**Test names + assertions:**
- `test_list_decodesPage` — 200 + paginated JSON → EarningsPage with items + page metadata.
- `test_list_emptyPage_handled` — empty `items` array → no error, total reflects backend value.
- `test_list_passesPaginationQueryParams` — captured request has `?page=2&limit=20`.

**Done when:** all 3 tests green.

### Task 9.2: EarningsViewModel + EarningsView
**Files:**
- `KilatRunner/Features/Earnings/EarningsViewModel.swift` — `@Observable` with `earnings`, `isLoading`, `errorMessage`, `currentPage`, `hasMore`. Methods: `loadFirstPage()`, `loadNextPage()` (idempotent, no duplicate fetches in flight).
- `KilatRunner/Features/Earnings/EarningsView.swift` — List with infinite scroll. Each row: date + booking ID short + amount.
- `KilatRunnerTests/Features/EarningsViewModelTests.swift`

**Test names + assertions:**
- `test_loadFirstPage_populatesEarnings` — repo returns page → VM earnings set, currentPage = 1.
- `test_loadNextPage_appendsItems_incrementsPage` — second page → items appended, page = 2.
- `test_loadNextPage_whileLoading_isIdempotent` — calling twice in quick succession only fires one repo call.
- `test_loadFirstPage_failure_setsError` — repo throws → errorMessage set.
- `test_hasMore_false_whenLastPageReached` — total items reached → `hasMore = false` blocks further loadNextPage.

**Done when:** all 5 tests green + manual smoke: navigate to Earnings from Dashboard → see list (will show the booking completed during Phase 8 smoke + any historical seeded data).

**Phase 9 push checkpoint** ✅

---

## Phase 10 — Smoke + polish

**Goal:** Manual smoke checklist passes end-to-end on a real device. README + screenshots updated. Optional first TestFlight build.

### Task 10.1: Manual smoke checklist doc
**Files:** create `app-runner-ios/docs/manual-smoke.md`
**Intent:** Codify the §13 Definition of Done from the spec as a 10-item checklist with explicit setup steps (which test owner account to use, how to seed a fresh booking via Bruno, expected backend log lines to look for).
**Done when:** doc reads as a clean checklist a fresh engineer could execute without asking questions.

### Task 10.2: Run full smoke on real device
**Files:** none — log results inline in `docs/manual-smoke.md` (date + pass/fail per item + bug notes).
**Done when:** all 10 items pass on a real iPhone. Any failure → file as task in a new "Phase 10 bug fix" backlog at the bottom of this plan and fix before declaring MVP done.

### Task 10.3: README + screenshots + final commit
**Files:** modify `app-runner-ios/README.md`
**Intent:** Add screenshots (login, dashboard, available jobs, active delivery map, earnings), build instructions, architecture diagram link to spec, "MVP complete" banner. Update Flutter `app-runner` README with a one-line pointer: "Frozen as reference for native rewrite — see `app-runner-ios` and `app-runner-android`."
**Done when:** README looks portfolio-ready on GitHub.

### Task 10.4 (optional): First TestFlight build
**Files:** none — Xcode + App Store Connect work.
**Intent:** Apple Developer enrolled (from Phase 0 Task 0.1). Archive build → upload → TestFlight internal track. Install on a real device via TestFlight.
**Done when:** TestFlight build installs and runs on a real iPhone that's not the dev machine.

**Phase 10 push checkpoint** ✅ — **iOS MVP complete.** 🎉

---

## Self-review notes

**Spec coverage:** every spec section maps to at least one task —
- §3 MVP screen list (6 screens) → Tasks 2.3, 3.3, 4.2, 4.3, 5.3, 9.2
- §4 Architecture → Phase 1 (Network), Phase 6/7 (Service), throughout (ViewModel/View/Repository pattern)
- §5 iOS module structure → Phase 0 Task 0.4 + structure block at top of this plan
- §7 Critical flows → Phases 2, 3, 4, 6, 7, 8 (one phase per flow)
- §8 Auth + refresh → Task 1.7
- §9 Background location → Phase 6
- §10 WebSocket → Phase 7
- §11 Error handling → covered per-task via ViewModel error tests
- §12 Testing strategy → TDD throughout; UI tests explicitly skipped per spec
- §13 Definition of done → Phase 10 Task 10.1 + 10.2
- §14 Repo / git layout → Phase 0 Task 0.5

**Out-of-MVP backlog** is in spec §15 — explicitly not in this plan.

**Plan format compliance:** No Swift code blocks in tasks (per `feedback_plan_format.md`). Bash/curl/gh commands kept because they're executable plan steps, not duplicate work.

---

## Execution options

**1. Subagent-Driven (recommended for portfolio learning):** I dispatch a fresh subagent per task; you review between tasks; tight loop, max learning.

**2. Inline Execution:** Execute tasks in this session using `superpowers:executing-plans`. Batch with checkpoints.

**3. Self-paced (you drive, I assist):** You execute tasks yourself in Xcode; ask me when stuck. Best for hands-on Swift learning.

Tell me which path and I'll set up the next step.
