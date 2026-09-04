# Phase 5: Performance Optimization ⚡

> **Objective:** Identify performance bottlenecks through static code analysis. Find code
> that causes jank, memory leaks, excessive battery drain, large app size, and slow startup.
> Mobile users have ZERO patience — 53% abandon apps that take longer than 3 seconds to load.

---

## 📋 PERFORMANCE CHECKS

### CHECK P1: Memory Leak Detection
```
WHAT TO CHECK:
  Memory leaks in mobile apps are the #1 performance killer. They cause:
  - App slowdown over time
  - OutOfMemory crashes
  - System killing the app in background

COMMON LEAK PATTERNS BY FRAMEWORK:

  Flutter:
  ❑ StreamController not closed in dispose()
  ❑ AnimationController not disposed
  ❑ TextEditingController not disposed
  ❑ FocusNode not disposed
  ❑ ScrollController not disposed
  ❑ Timer not cancelled in dispose()
  ❑ Listener added but never removed
  ❑ ChangeNotifier not disposed
  ❑ GlobalKey holding references to disposed widgets

  React Native:
  ❑ useEffect without cleanup function
  ❑ setInterval/setTimeout not cleared
  ❑ Event listeners not removed on unmount
  ❑ Subscriptions not unsubscribed
  ❑ Ref holding stale component references
  ❑ Context causing unnecessary re-renders of large subtrees

  Kotlin/Android Native:
  ❑ Activity/Fragment reference held by long-lived object
  ❑ Handler.postDelayed without removeCallbacks
  ❑ Non-static inner class holding Activity reference
  ❑ BroadcastReceiver not unregistered
  ❑ Cursor not closed
  ❑ InputStream/OutputStream not closed
  ❑ ViewBinding not nullified in Fragment.onDestroyView

  Swift/iOS Native:
  ❑ Strong reference cycles between classes (should use [weak self])
  ❑ Closure capturing self strongly
  ❑ NotificationCenter observer not removed
  ❑ Timer not invalidated
  ❑ Delegate not declared as weak
  ❑ Core Data context not released

SEARCH PATTERNS:
  Flutter: Find every class extending State, check for dispose() method
  RN: Find every useEffect, check for return cleanup
  Android: Find every onStart/onResume, check for matching onStop/onPause
  iOS: Find every addObserver, check for matching removeObserver

CITATION REQUIRED: List EVERY disposable resource and its cleanup status
```

### CHECK P2: Unnecessary Widget Rebuilds / Re-renders
```
WHAT TO CHECK:
  Every rebuild/re-render costs CPU cycles and battery. The UI should
  only rebuild what ACTUALLY changed.

FLUTTER-SPECIFIC:
  ❑ Are const constructors used where possible?
  ❑ Is setState() called too broadly? (rebuilding entire tree for one change)
  ❑ Are BLoC/Provider selectors used to limit rebuilds?
  ❑ Are large lists using RepaintBoundary?
  ❑ Is shouldRebuild/buildWhen used to filter unnecessary updates?
  ❑ Are animations not causing full tree rebuilds?

  RED FLAGS:
  🔴 setState(() { }) called in a parent widget for child-only change
  🔴 BlocBuilder without buildWhen (rebuilds on every state change)
  🔴 Consumer wrapping the entire screen instead of specific widget
  🔴 StreamBuilder wrapping large widget tree
  🔴 ValueNotifier without ValueListenableBuilder (full rebuild)

REACT NATIVE-SPECIFIC:
  ❑ Are components wrapped in React.memo where appropriate?
  ❑ Is useMemo used for expensive computations?
  ❑ Is useCallback used for callback props?
  ❑ Are inline function props avoided? (creates new function each render)
  ❑ Is context split to avoid unnecessary re-renders?

  RED FLAGS:
  🔴 Component re-renders on parent state change it doesn't use
  🔴 New object/array created in render: style={{...}} on every render
  🔴 Context provider wrapping entire app with frequently changing value
  🔴 Inline arrow functions as props: onPress={() => doSomething()}

COMPOSE-SPECIFIC:
  ❑ Are unstable parameters avoided in Composables?
  ❑ Is remember() used for expensive objects?
  ❑ Are lambdas stable? (not recreated on every recomposition)
  ❑ Is derivedStateOf() used where appropriate?

CITATION REQUIRED: Find the 3 worst rebuild offenders and explain the fix
```

### CHECK P3: Image & Asset Optimization
```
WHAT TO CHECK:
  ❑ What is the total app size? (assets contribution)
  ❑ Are images compressed? (not raw bitmap/PNG for photos)
  ❑ Are appropriate formats used? (WebP, SVG for icons, compressed JPEG for photos)
  ❑ Are images sized appropriately? (not loading 4000x3000 for 200x200 display)
  ❑ Is lazy loading implemented for off-screen images?
  ❑ Is image caching configured? (CachedNetworkImage, FastImage)
  ❑ Are unused assets present? (increasing app size for nothing)
  ❑ Are fonts subsetted? (not loading entire font with 10,000 glyphs)

SIZE BENCHMARKS:
  - App icon: < 1MB total (all resolutions)
  - Individual image asset: < 500KB
  - Total assets folder: < 20MB
  - Individual font file: < 500KB
  - Total app download size target: < 50MB

OPTIMIZATION OPPORTUNITIES:
  🔵 PNG screenshots → convert to WebP (50-70% smaller)
  🔵 Unused resolution variants → remove unused densities
  🔵 Full font files → subset to used characters only
  🔵 Inline SVGs → extract to asset files
  🔵 Animated GIFs → convert to Lottie animations

CITATION REQUIRED: List largest assets and recommend optimizations
```

### CHECK P4: Main Thread / UI Thread Blocking
```
WHAT TO CHECK:
  The main thread renders UI at 60fps (16ms per frame). Any operation
  taking longer than 16ms causes visible jank.

OPERATIONS THAT MUST BE OFF MAIN THREAD:
  ❑ File I/O (reading/writing files)
  ❑ Database queries (SQLite, Realm, Hive)
  ❑ Network requests (HTTP calls)
  ❑ Image processing (resize, compress, filter)
  ❑ JSON parsing of large payloads
  ❑ Encryption/decryption
  ❑ Complex calculations (sorting large lists, algorithms)
  ❑ Shared preferences access for large data

SEARCH PATTERNS:
  Flutter:
    - compute() or Isolate for heavy work?
    - await on main isolate for heavy computation?
    - rootBundle.load() for large assets?

  React Native:
    - InteractionManager.runAfterInteractions() used?
    - Heavy computation in render path?
    - Bridge crossing with large data?

  Android:
    - Operations on main thread? (StrictMode would catch)
    - Room queries without suspend/Flow?
    - Bitmap operations without background thread?

  iOS:
    - DispatchQueue.main for heavy work?
    - Core Data fetch on main thread?
    - Image processing without background queue?

CITATION REQUIRED: Show any main-thread-blocking operations found
```

### CHECK P5: Network Efficiency
```
WHAT TO CHECK:
  ❑ Are API responses cached appropriately?
  ❑ Is data pagination implemented? (not fetching all records)
  ❑ Are images lazy-loaded and cached?
  ❑ Is there request deduplication? (same request fired twice)
  ❑ Are WebSocket connections managed efficiently?
  ❑ Is data compression enabled? (gzip)
  ❑ Are unnecessary API calls eliminated?
  ❑ Is there a data freshness strategy? (when to refresh vs use cache)

RED FLAGS:
  🔴 API call made on every widget build/render (not just once)
  🔴 Same API called from multiple places simultaneously
  🔴 Full user profile fetched when only name is needed
  🔴 Polling every second when WebSocket/push would be better
  🔴 Downloading full image when thumbnail is sufficient
  🔴 No caching — identical requests repeat on every screen visit
  🔴 API response of 10MB when 100KB would suffice

CITATION REQUIRED: Map all API calls and their frequency/caching strategy
```

### CHECK P6: Startup Time Optimization
```
WHAT TO CHECK:
  ❑ What happens during app startup? (trace the initialization sequence)
  ❑ Are there blocking operations before first frame?
  ❑ Is lazy initialization used for non-critical services?
  ❑ Is the splash screen covering actual loading or just a delay?
  ❑ Are shared preferences read synchronously during startup?
  ❑ Are analytics/crash reporting SDKs initialized efficiently?

STARTUP BUDGET:
  Cold start target: < 2 seconds to interactive
  Warm start target: < 1 second to interactive
  Hot start target: < 0.5 seconds to interactive

RED FLAGS:
  🔴 Fetching data from network before showing any UI
  🔴 Initializing 10+ services sequentially in main()
  🔴 Loading entire database into memory on startup
  🔴 Synchronous shared preferences read in main()
  🔴 Heavy computation in initState() / init()
  🔴 Artificial delay on splash screen (Timer.delayed just for aesthetics)

CITATION REQUIRED: Show the startup sequence and identify blocking operations
```

### CHECK P7: Battery Consumption
```
WHAT TO CHECK:
  ❑ Is location tracking efficient? (not requesting GPS every second)
  ❑ Are background tasks optimized? (not running continuously)
  ❑ Is the screen kept awake unnecessarily? (WakeLock)
  ❑ Are sensors used efficiently? (accelerometer, gyroscope)
  ❑ Are timers/intervals cleaned up?
  ❑ Is Bluetooth/NFC used efficiently?

BATTERY KILLERS:
  🔴 Location updates every 1 second (should be every 30s+ for most apps)
  🔴 Background service running continuously for polling
  🔴 Keeping screen awake on all screens (not just video/navigation)
  🔴 Continuous animations on screens that are not visible
  🔴 Wake locks not released after use
  🔴 Unnecessary sensor polling

CITATION REQUIRED: Find all location, timer, and background processing code
```

### CHECK P8: Database Performance
```
WHAT TO CHECK:
  ❑ Are database queries indexed? (WHERE clause columns)
  ❑ Are joins optimized? (not N+1 query problem)
  ❑ Is batch processing used for multiple inserts?
  ❑ Are large result sets paginated?
  ❑ Is the database connection properly managed?
  ❑ Are migrations handled correctly?
  ❑ Is database size monitored?

COMMON ISSUES:
  🐛 SELECT * FROM orders → fetches all columns when only 2 are needed
  🐛 Loop with individual queries → should be batch query
  🐛 Missing index on column used in WHERE clause
  🐛 JOIN without index → full table scan
  🐛 Database migration fails → app crashes on update
  🐛 Cursor/connection not closed → resource leak
  🐛 Transaction not used for multiple related writes → data inconsistency

CITATION REQUIRED: Show all database queries and assess performance
```

---

## 🚦 PHASE 5 GATE — MANDATORY CHECKLIST

```
PHASE 5 GATE CHECKLIST:
  □ [P1] Memory leak audit completed (all disposables checked)
  □ [P2] Unnecessary rebuild/re-render analysis done
  □ [P3] Image and asset optimization reviewed
  □ [P4] Main thread blocking operations identified
  □ [P5] Network efficiency assessed
  □ [P6] Startup time sequence analyzed
  □ [P7] Battery consumption patterns checked
  □ [P8] Database query performance reviewed
  □ Minimum 6 code citations provided
  □ Files examined list produced
  □ Cross-reference to Phase 2 UI issues (list performance)
```

### Gate Report Format:
```
═════════════════════════════════════════════════════════════
  ✅ PHASE 5 COMPLETE: Performance Analysis
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/11 items)

  ⚡ Performance Health Score: [X/10]
  Memory Leaks Found: [count]
  Unnecessary Rebuilds: [count]
  Main Thread Violations: [count]
  Estimated App Size Impact: [size]
  Startup Sequence: [Fast/Moderate/Slow]
═════════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present performance findings before proceeding to Phase 6.**
