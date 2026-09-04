# Phase 7: API & Network Resilience 🌐

> **Objective:** Audit every network call in the app for correctness, error handling,
> offline behavior, caching strategy, and resilience. Mobile apps operate in hostile
> network environments — intermittent connectivity, slow connections, timeouts, and
> unexpected server responses are the NORM, not the exception.

---

## 🎯 WHY THIS PHASE MATTERS

Mobile users are on subways, elevators, rural areas, and switching between WiFi and
cellular constantly. An app that only works with perfect internet is a broken app.

---

## 📋 API & NETWORK CHECKS

### CHECK N1: HTTP Client Configuration
```
WHAT TO CHECK:
  ❑ Is there a centralized HTTP client? (not creating new instances everywhere)
  ❑ Are timeouts configured?
    - Connection timeout: 10-30 seconds
    - Read timeout: 30-60 seconds
    - Write timeout: 30-60 seconds
  ❑ Are interceptors/middleware set up for:
    - Authentication header injection
    - Logging (debug only)
    - Error transformation
    - Token refresh on 401
    - Retry logic
  ❑ Is the base URL configurable per environment?
  ❑ Is certificate pinning configured? (see Security Phase)
  ❑ Is request/response compression enabled?

FRAMEWORK-SPECIFIC:
  Flutter (Dio/http):
    ❑ Dio interceptors configured?
    ❑ BaseOptions with timeout set?
    ❑ CancelToken supported for in-flight requests?

  React Native (Axios/fetch):
    ❑ Axios instance with defaults?
    ❑ Request/response interceptors?
    ❑ AbortController for cancellation?

  Kotlin (Retrofit/OkHttp):
    ❑ OkHttpClient with timeouts?
    ❑ Interceptor chain configured?
    ❑ Connection pooling configured?

  Swift (URLSession/Alamofire):
    ❑ URLSessionConfiguration with timeouts?
    ❑ Request adapters/retriers?
    ❑ Session delegate configured?

RED FLAGS:
  🔴 No timeout → request hangs forever
  🔴 New HTTP client created per request → resource waste
  🔴 No interceptor for auth → token manually added everywhere
  🔴 Logging interceptor active in release → leaks sensitive data
  🔴 Base URL hardcoded in every request → impossible to change

CITATION REQUIRED: Show HTTP client setup and configuration
```

### CHECK N2: API Error Handling
```
WHAT TO CHECK:
  For EVERY API call, verify handling of these scenarios:

  HTTP STATUS CODES:
  ❑ 200 OK → Data parsed correctly
  ❑ 201 Created → Resource creation confirmed
  ❑ 204 No Content → Empty body handled (no JSON parse error)
  ❑ 400 Bad Request → Validation errors shown to user
  ❑ 401 Unauthorized → Token refresh triggered OR redirect to login
  ❑ 403 Forbidden → Permission denied message shown
  ❑ 404 Not Found → "Item not found" message, not crash
  ❑ 409 Conflict → Conflict resolution (e.g., stale data)
  ❑ 422 Unprocessable → Field-level errors mapped to form
  ❑ 429 Too Many Requests → Retry with backoff
  ❑ 500 Internal Server Error → Generic "try again later" message
  ❑ 502/503 Service Unavailable → Retry with backoff
  ❑ Network Error (no internet) → Offline message, cached data shown

  COMMON BUGS:
  🐛 Only 200 is handled → any other status crashes the app
  🐛 401 triggers login redirect but loses current user action
  🐛 Error response has different JSON structure → crash on parse
  🐛 Generic "Something went wrong" for ALL errors (unhelpful)
  🐛 No error at all → user stares at loading spinner forever
  🐛 Error message from server shown raw to user (technical jargon)
  🐛 Network timeout shows "No internet" (misleading — internet exists, server slow)
  🐛 Retry on error without exponential backoff → floods server

ERROR HANDLING PATTERN TO VERIFY:
  Every API call should follow:
  try {
    response = await apiCall()
    if (response.success) → handle data
    else → handle business error
  } catch (NetworkError) → handle offline
  catch (TimeoutError) → handle timeout
  catch (ParseError) → handle malformed response
  catch (Unknown) → handle unexpected error
  finally → hide loading state

CITATION REQUIRED: Show error handling for at least 5 different API calls
```

### CHECK N3: Offline Mode & Connectivity
```
WHAT TO CHECK:
  ❑ Does the app detect connectivity changes? (online ↔ offline)
  ❑ What happens when the app goes offline?
    Option A: Show cached data with "offline" indicator ✅
    Option B: Show empty screen with "no internet" ✅
    Option C: App crashes or shows infinite loading ❌
  ❑ Are user actions queued when offline? (submitted when back online)
  ❑ Is there a sync mechanism for offline data?
  ❑ Does the app recover gracefully when connectivity returns?
  ❑ Is the connectivity indicator accurate? (not false positives)

OFFLINE STRATEGY ASSESSMENT:
  Level 0: No offline support (crashes without internet) ❌
  Level 1: Shows "no internet" message, app unusable offline ⚠️
  Level 2: Shows cached data, read-only mode offline ✅
  Level 3: Full offline support with sync queue ✅✅

COMMON BUGS:
  🐛 App checks connectivity once at startup, never again
  🐛 Connectivity check uses ping (fails on captive portals)
  🐛 Offline actions lost on app restart (not persisted)
  🐛 Sync conflict: offline edit + server edit = data loss
  🐛 App shows "no internet" on slow but connected network
  🐛 Back online → app doesn't refresh, shows stale data
  🐛 Queued actions replayed in wrong order
  🐛 Large offline queue overflows storage

CITATION REQUIRED: Show connectivity detection and offline handling strategy
```

### CHECK N4: Request Retry & Backoff Strategy
```
WHAT TO CHECK:
  ❑ Are failed requests retried automatically?
  ❑ Is exponential backoff implemented? (not instant retry)
  ❑ Is there a maximum retry count?
  ❑ Are only idempotent requests retried? (GET yes, POST maybe not)
  ❑ Is the user informed of retry attempts?
  ❑ Can the user manually retry?
  ❑ Are retries cancelled when user navigates away?

CORRECT RETRY PATTERN:
  Attempt 1: Immediate
  Attempt 2: Wait 1 second
  Attempt 3: Wait 2 seconds
  Attempt 4: Wait 4 seconds
  Attempt 5: Give up, show error with manual retry option

  With jitter: Add random 0-500ms to prevent thundering herd

DANGEROUS PATTERNS:
  🔴 while(true) { retry() } → infinite retry loop
  🔴 Retry POST request → duplicate order/payment
  🔴 Retry without delay → DDoS own server
  🔴 No maximum retries → user stuck in retry loop
  🔴 Retry on 400 Bad Request → will never succeed
  🔴 Retry without backoff → wastes battery on hopeless retries

WHICH ERRORS SHOULD BE RETRIED:
  ✅ Retry: Network error, timeout, 500, 502, 503, 429
  ❌ Don't retry: 400, 401, 403, 404, 409, 422

CITATION REQUIRED: Show retry logic implementation (or note its absence)
```

### CHECK N5: Data Caching Strategy
```
WHAT TO CHECK:
  ❑ What data is cached locally?
  ❑ How long is the cache valid? (TTL - Time To Live)
  ❑ Is the cache invalidated correctly? (after update/delete)
  ❑ Is the cache size bounded? (doesn't grow infinitely)
  ❑ Is sensitive data excluded from cache?
  ❑ Is cache cleared on logout?
  ❑ Is stale-while-revalidate implemented? (show cached, fetch fresh)

CACHING STRATEGIES:
  1. Cache-First: Show cached data, then fetch and update
     - Best for: lists, profiles, settings
  2. Network-First: Try network, fall back to cache
     - Best for: real-time data, financial info
  3. Cache-Only: Only show cached data
     - Best for: offline mode
  4. Network-Only: Always fetch fresh
     - Best for: one-time verifications, payment status

WHAT SHOULD BE CACHED:
  ✅ User profile data (changes rarely)
  ✅ Product listings (with TTL)
  ✅ Images and thumbnails (with size limit)
  ✅ Configuration/feature flags
  ✅ Recent search results
  ❌ Payment transactions (always fetch fresh)
  ❌ Real-time chat messages (use WebSocket)
  ❌ Sensitive tokens (use secure storage)

COMMON BUGS:
  🐛 No caching at all → every screen visit hits the network
  🐛 Cache never expires → shows outdated data forever
  🐛 Cache not invalidated after user edits data
  🐛 Cache grows unbounded → fills device storage
  🐛 Sensitive data in cache → accessible after logout
  🐛 Cache key collision → wrong data shown

CITATION REQUIRED: Map all cached data, its TTL, and invalidation triggers
```

### CHECK N6: File Upload & Download
```
WHAT TO CHECK:
  ❑ Are uploads multipart-encoded correctly?
  ❑ Is there progress indication for large uploads?
  ❑ Are uploads resumable after interruption?
  ❑ Is there file size validation before upload?
  ❑ Is file type validation before upload?
  ❑ Are downloads saved to correct location?
  ❑ Is download progress shown?
  ❑ Are partial downloads handled? (resume from where stopped)
  ❑ Is storage permission requested for downloads?

COMMON BUGS:
  🐛 Large file upload with no progress bar → user thinks app froze
  🐛 Upload timeout for large files (timeout too short)
  🐛 File picker returns null → crash (user cancelled)
  🐛 Image upload sends original 10MB photo instead of compressed
  🐛 Download fails silently → no error message
  🐛 Downloaded file has wrong extension or can't be opened
  🐛 Multiple upload requests for same file (retry without check)

CITATION REQUIRED: Show file upload/download implementation
```

### CHECK N7: WebSocket & Real-time Communication
```
WHAT TO CHECK (if applicable):
  ❑ Is WebSocket connection managed properly? (connect/disconnect lifecycle)
  ❑ Is reconnection implemented with backoff?
  ❑ Is the connection closed when app goes to background?
  ❑ Are messages queued during reconnection?
  ❑ Is heartbeat/ping-pong implemented?
  ❑ Are large payloads handled efficiently?
  ❑ Is the connection over WSS (not WS)?
  ❑ Are message parsing errors handled?

COMMON BUGS:
  🐛 WebSocket not reconnecting after network change
  🐛 Messages lost during reconnection
  🐛 Connection not closed on logout → receives other user's messages
  🐛 Memory leak from accumulated WebSocket messages
  🐛 No heartbeat → connection dies silently
  🐛 Large message blocks the UI thread
  🐛 Multiple WebSocket connections opened accidentally

CITATION REQUIRED: Show WebSocket lifecycle management (if applicable)
```

### CHECK N8: API Request Cancellation
```
WHAT TO CHECK:
  ❑ Are in-flight requests cancelled when user navigates away?
  ❑ Are search requests cancelled when user types new query?
  ❑ Are duplicate simultaneous requests prevented?
  ❑ Is cancellation token/controller supported?

COMMON BUGS:
  🐛 User navigates away → response arrives → setState on disposed widget
  🐛 Typing fast in search → 10 requests fire, responses arrive out of order
  🐛 Opening and closing a screen rapidly → response updates wrong instance
  🐛 Cancel button doesn't actually cancel the request (just hides UI)

FRAMEWORK-SPECIFIC:
  Flutter/Dio: CancelToken
  React Native/Axios: AbortController
  Kotlin/Retrofit: Call.cancel()
  Swift/URLSession: URLSessionTask.cancel()

CITATION REQUIRED: Show request cancellation handling (or note its absence)
```

---

## 🚦 PHASE 7 GATE — MANDATORY CHECKLIST

```
PHASE 7 GATE CHECKLIST:
  □ [N1] HTTP client configuration reviewed
  □ [N2] API error handling verified for all status codes
  □ [N3] Offline mode and connectivity detection checked
  □ [N4] Retry and backoff strategy verified
  □ [N5] Caching strategy mapped and assessed
  □ [N6] File upload/download handling checked
  □ [N7] WebSocket/real-time communication reviewed (if applicable)
  □ [N8] Request cancellation verified
  □ Minimum 6 code citations provided
  □ Files examined list produced
  □ API endpoint inventory produced
```

### API Endpoint Inventory (MANDATORY):
```
┌────┬──────────┬──────────────────┬──────────┬───────────┬──────────┐
│ #  │ Method   │ Endpoint         │ Cached?  │ Error     │ Retry?   │
│    │          │                  │          │ Handled?  │          │
├────┼──────────┼──────────────────┼──────────┼───────────┼──────────┤
│ 1  │ GET      │ /api/users/me    │ Yes/No   │ Yes/No    │ Yes/No   │
│ 2  │ POST     │ /api/orders      │ N/A      │ Yes/No    │ Yes/No   │
│ ...│ ...      │ ...              │ ...      │ ...       │ ...      │
└────┴──────────┴──────────────────┴──────────┴───────────┴──────────┘
```

### Gate Report Format:
```
══════════════════════════════════════════════════════════
  ✅ PHASE 7 COMPLETE: API & Network Resilience
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/11 items)

  🌐 Network Health Score: [X/10]
  Total API Endpoints: [count]
  Error Handling Coverage: [X]%
  Offline Support Level: [0-3]
  Caching Strategy: [None/Partial/Comprehensive]
══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present network findings before proceeding to Phase 8.**
