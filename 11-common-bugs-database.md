# Common Mobile App Bugs Database 🐛

> **Reference document containing 200+ real-world mobile app bugs categorized by type,
> framework, and severity. Use this as a lookup table during each review phase.**

---

## 🔴 CRITICAL BUGS (App-Breaking / Security Breach)

### Authentication & Security
```
BUG-C001: Hardcoded API key in source code
  Frameworks: ALL
  Pattern: const API_KEY = "sk_live_xxxxx"
  Impact: Full API access to attackers who decompile the app
  Fix: Use environment variables, server-side proxy, or secure key store

BUG-C002: Auth token stored in SharedPreferences/UserDefaults (unencrypted)
  Frameworks: Flutter, React Native, Android, iOS
  Pattern: prefs.setString('token', jwt) / AsyncStorage.setItem('token', jwt)
  Impact: Any app with root access can steal user sessions
  Fix: Use flutter_secure_storage / Keychain / Android Keystore

BUG-C003: HTTP used instead of HTTPS
  Frameworks: ALL
  Pattern: baseUrl = "http://api.example.com"
  Impact: All data interceptable on public WiFi (MITM attack)
  Fix: Enforce HTTPS everywhere, configure ATS/network_security_config

BUG-C004: SQL injection in local database
  Frameworks: ALL with local DB
  Pattern: db.rawQuery("SELECT * FROM users WHERE name = '$input'")
  Impact: Data theft or deletion from local database
  Fix: Use parameterized queries: db.query("users", where: "name = ?", whereArgs: [input])

BUG-C005: Debug mode enabled in production release
  Frameworks: ALL
  Pattern: android:debuggable="true" / kDebugMode checks missing
  Impact: App can be debugged, memory inspected, data extracted
  Fix: Ensure release builds have debug disabled

BUG-C006: Certificate validation disabled
  Frameworks: ALL
  Pattern: badCertificateCallback: (cert, host, port) => true
  Impact: MITM attacks succeed even on HTTPS
  Fix: Remove bypass, implement proper certificate pinning

BUG-C007: Password logged to console
  Frameworks: ALL
  Pattern: print("Login: $email, $password") / console.log(credentials)
  Impact: Password visible in device logs accessible by other apps
  Fix: Never log sensitive data, use log levels properly

BUG-C008: Biometric bypass with local flag
  Frameworks: ALL
  Pattern: if (biometricPassed) { prefs.setBool('authenticated', true) }
  Impact: Attacker modifies local storage to bypass authentication
  Fix: Biometric must unlock server-side token, not set local flag

BUG-C009: WebView JavaScript injection
  Frameworks: ALL with WebView
  Pattern: webView.loadUrl("javascript:$userInput")
  Impact: Arbitrary code execution in WebView context
  Fix: Sanitize all input, use postMessage for communication

BUG-C010: Unencrypted sensitive database
  Frameworks: ALL with local DB
  Pattern: SQLite/Hive/Realm without encryption for user data
  Impact: All user data readable if device is compromised
  Fix: Use SQLCipher / encrypted Hive / encrypted Realm
```

### App Crashes
```
BUG-C011: Force unwrap on null API response
  Frameworks: Flutter/Dart, Swift, Kotlin
  Pattern: response.data!['user']!['name'] / response!!.user!!.name
  Impact: App crashes when API returns unexpected null
  Fix: Use null-safe access with fallback: response?.data?['user']?['name'] ?? ''

BUG-C012: Missing database migration
  Frameworks: ALL with local DB
  Pattern: Schema changed but no migration defined for version N to N+1
  Impact: App crashes on update for ALL existing users
  Fix: Define explicit migration for every schema change

BUG-C013: Unhandled platform exception in plugin
  Frameworks: Flutter, React Native
  Pattern: Platform channel call without try-catch / native module without error handling
  Impact: App crashes when native feature unavailable
  Fix: Wrap all platform calls in try-catch with PlatformException handling

BUG-C014: Stack overflow from infinite recursion
  Frameworks: ALL
  Pattern: Function calls itself without proper base case
  Impact: App crashes with stack overflow
  Fix: Add base case, convert to iteration, or limit recursion depth

BUG-C015: Out of memory from unbounded list/cache
  Frameworks: ALL
  Pattern: List grows without limit, no cache eviction
  Impact: App killed by OS when memory exhausted
  Fix: Implement max size, LRU cache eviction, pagination
```

---

## 🟠 HIGH SEVERITY BUGS (Major Functionality Issues)

### UI / Navigation
```
BUG-H001: Button with empty onPressed handler
  Frameworks: Flutter
  Pattern: ElevatedButton(onPressed: () {}, child: Text('Submit'))
  Impact: User taps button, nothing happens, no feedback
  Fix: Implement handler or show "coming soon" message

BUG-H002: Keyboard covers input field
  Frameworks: ALL
  Pattern: Form inputs at bottom of screen without scroll adjustment
  Impact: User can't see what they're typing
  Fix: Wrap in SingleChildScrollView, use resizeToAvoidBottomInset

BUG-H003: Back button closes app instead of navigating back
  Frameworks: Flutter, React Native
  Pattern: WillPopScope / BackHandler not implemented on nested screens
  Impact: User loses all progress when pressing back
  Fix: Handle back navigation properly per screen

BUG-H004: Infinite loading on error
  Frameworks: ALL
  Pattern: Loading state set to true, never set to false on error
  Impact: User sees spinner forever, can't interact with app
  Fix: Set loading=false in finally block or catch block

BUG-H005: Navigation stack grows infinitely
  Frameworks: ALL
  Pattern: Navigator.push used repeatedly instead of pushReplacement
  Impact: Memory grows, back button goes through 50 screens
  Fix: Use pushReplacement for login→home, pushNamedAndRemoveUntil for logout

BUG-H006: Double form submission
  Frameworks: ALL
  Pattern: No loading state to prevent multiple taps on submit button
  Impact: Duplicate orders, payments, registrations
  Fix: Disable button during submission, use loading flag

BUG-H007: List shows duplicates after refresh
  Frameworks: ALL
  Pattern: New data appended to existing list instead of replacing
  Impact: Same items appear multiple times
  Fix: Clear list before adding refreshed data, or use set-based deduplication

BUG-H008: Screen rotation loses state
  Frameworks: Android Native, React Native
  Pattern: State not saved in onSaveInstanceState / no state preservation
  Impact: Form data, scroll position, selections all lost
  Fix: Save state properly, use ViewModel (Android), or lock orientation

BUG-H009: Tab bar state lost on tab switch
  Frameworks: ALL
  Pattern: Tab content recreated on every tab switch
  Impact: User loses scroll position, typed text, selections
  Fix: Use AutomaticKeepAliveClientMixin (Flutter), keep tab state alive

BUG-H010: Modal can't be dismissed
  Frameworks: ALL
  Pattern: Modal/dialog with no close button and no gesture dismissal
  Impact: User is trapped, must kill app
  Fix: Add close button, enable tap-outside-to-dismiss, enable swipe-to-dismiss
```

### Logic & Data
```
BUG-H011: Price calculation with floating point error
  Frameworks: ALL
  Pattern: double total = 19.99 * 3; // = 59.96999999999
  Impact: Wrong prices shown, financial calculation errors
  Fix: Use Decimal type, or calculate in cents (integers)

BUG-H012: Timezone-unaware date comparison
  Frameworks: ALL
  Pattern: if (eventDate.isBefore(DateTime.now()))
  Impact: Events appear expired or available depending on timezone
  Fix: Normalize all dates to UTC before comparison

BUG-H013: Pagination off-by-one error
  Frameworks: ALL
  Pattern: page starts at 0 but API expects 1, or vice versa
  Impact: First page loaded twice, or first page data missed
  Fix: Verify page numbering matches API expectation

BUG-H014: Search returns results for previous query
  Frameworks: ALL
  Pattern: Async search without canceling previous request
  Impact: Results for "app" shown when user searched "apple"
  Fix: Cancel previous request when new query starts, use debounce

BUG-H015: Logout doesn't clear local data
  Frameworks: ALL
  Pattern: Logout clears token but not cached user data / database
  Impact: Next user sees previous user's data
  Fix: Clear ALL local storage on logout: DB, cache, preferences, files

BUG-H016: Optimistic update without rollback
  Frameworks: ALL
  Pattern: UI updated before API confirms, but no undo on failure
  Impact: UI shows success but server has different data
  Fix: Implement rollback on API failure, show error and revert state

BUG-H017: Token refresh infinite loop
  Frameworks: ALL
  Pattern: 401 → refresh token → 401 → refresh → 401 → ...
  Impact: App floods server with refresh requests, battery drain
  Fix: Limit refresh attempts, logout after N failures, check refresh token expiry

BUG-H018: Race condition on rapid button taps
  Frameworks: ALL
  Pattern: API call triggered per tap without debounce or lock
  Impact: Multiple identical requests, duplicate data/actions
  Fix: Add loading lock, debounce taps, disable button during processing

BUG-H019: Form validation only on submit
  Frameworks: ALL
  Pattern: No real-time validation, errors only shown after submit attempt
  Impact: User fills entire form, submits, then discovers first field is wrong
  Fix: Validate on field change or field blur, show inline errors

BUG-H020: Deep link crashes on invalid parameter
  Frameworks: ALL
  Pattern: myapp://product?id=INVALID → int.parse("INVALID") → crash
  Impact: App crashes from external link (marketing email, notification)
  Fix: Validate all deep link parameters, show error for invalid ones
```

---

## 🟡 MEDIUM SEVERITY BUGS (UX Degradation / Minor Issues)

### UI/UX
```
BUG-M001: Text overflow / clipping on long content
  Pattern: Text widget without maxLines or overflow property
  Fix: Add overflow: TextOverflow.ellipsis, maxLines: 2

BUG-M002: Touch target too small (< 48x48 dp)
  Pattern: 24x24 icon without padding for touch area
  Fix: Wrap in SizedBox(width: 48, height: 48) or use hitSlop

BUG-M003: No empty state for lists
  Pattern: List shows blank white space when empty
  Fix: Show illustration + message like "No items yet. Add your first!"

BUG-M004: No loading skeleton / placeholder
  Pattern: Screen is blank while data loads
  Fix: Show skeleton loading animation matching expected layout

BUG-M005: Hardcoded colors instead of theme
  Pattern: Color(0xFF000000) instead of Theme.of(context).textTheme
  Fix: Use theme colors for all UI elements

BUG-M006: Missing pull-to-refresh
  Pattern: List/page with no way to refresh without leaving screen
  Fix: Add RefreshIndicator / pull-to-refresh gesture

BUG-M007: No visual feedback on tap
  Pattern: GestureDetector without InkWell/ripple effect
  Fix: Use InkWell (Flutter), TouchableOpacity (RN) for feedback

BUG-M008: Inconsistent spacing/padding across screens
  Pattern: Different padding values on different screens (16, 20, 12, 24)
  Fix: Define spacing constants and use consistently

BUG-M009: Images without error/loading placeholder
  Pattern: Image.network(url) with no placeholder or errorBuilder
  Fix: Add loading placeholder (shimmer) and error fallback (icon)

BUG-M010: No dark mode support
  Pattern: All colors hardcoded for light theme
  Fix: Implement ThemeData for both light and dark modes

BUG-M011: Keyboard type wrong for input field
  Pattern: Phone number field showing alphabetic keyboard
  Fix: Set keyboardType: TextInputType.phone for each field appropriately

BUG-M012: No scroll-to-top on tab re-tap
  Pattern: Tapping active tab does nothing
  Fix: Scroll content to top when user taps already-selected tab

BUG-M013: Snackbar/toast behind bottom navigation
  Pattern: Notification hidden behind bottom bar
  Fix: Add bottom margin equal to bottom bar height

BUG-M014: Status bar text invisible
  Pattern: Dark text on dark status bar background (or vice versa)
  Fix: Set SystemUiOverlayStyle matching screen background

BUG-M015: No haptic feedback on important actions
  Pattern: Destructive actions (delete, purchase) without tactile feedback
  Fix: Add HapticFeedback.heavyImpact() for important actions
```

### Performance
```
BUG-M016: ListView(children: [...]) with many items
  Pattern: All items built at once instead of lazily
  Fix: Use ListView.builder for dynamic/large lists

BUG-M017: Image not cached on network load
  Pattern: Image.network() re-downloads every time
  Fix: Use CachedNetworkImage (Flutter) / FastImage (RN)

BUG-M018: Full widget tree rebuild on single value change
  Pattern: setState() in parent rebuilds 50+ child widgets
  Fix: Use targeted state management (BLoC selector, Consumer, etc.)

BUG-M019: Large JSON parsed on main thread
  Pattern: jsonDecode of 5MB response blocking UI
  Fix: Use compute() / Isolate (Flutter), worker thread (native)

BUG-M020: Timer not canceled in dispose
  Pattern: Timer.periodic created but never canceled
  Fix: Store timer reference, cancel in dispose()

BUG-M021: Unnecessary API calls on widget rebuild
  Pattern: API called inside build() or render() method
  Fix: Call API in initState/useEffect/init, not in build

BUG-M022: Animation controller not disposed
  Pattern: AnimationController created but not disposed
  Fix: Add controller.dispose() in dispose() method

BUG-M023: No pagination on large data set
  Pattern: Fetching all 10,000 records at once
  Fix: Implement pagination with page size of 20-50

BUG-M024: Redundant network requests
  Pattern: Same API called from parent and child widget
  Fix: Fetch once at parent level, pass data down

BUG-M025: Synchronous file read on startup
  Pattern: File.readAsStringSync() in main()
  Fix: Use async version, or load after first frame renders
```

### State Management
```
BUG-M026: State not updated after background return
  Pattern: User leaves app, data changes, returns to stale data
  Fix: Refresh data on AppLifecycleState.resumed

BUG-M027: Stream subscription not cancelled
  Pattern: stream.listen() without storing and canceling subscription
  Fix: Store subscription, cancel in dispose/cleanup

BUG-M028: Provider/context used after dispose
  Pattern: Async callback accesses context after widget disposed
  Fix: Check mounted (Flutter), use ref.read not ref.watch in callbacks

BUG-M029: Multiple instances of same state container
  Pattern: BLoC/ViewModel created per screen visit instead of shared
  Fix: Scope to correct lifecycle (route-level, app-level)

BUG-M030: State persists across user sessions
  Pattern: Previous user's state visible after logout + new login
  Fix: Reset all state containers on logout
```

### Network
```
BUG-M031: No timeout on API request
  Pattern: HTTP request with no timeout configured
  Fix: Set connection timeout (15s) and read timeout (30s)

BUG-M032: No retry on transient failure
  Pattern: Network error shows "failed" with no retry option
  Fix: Add retry button, implement automatic retry with backoff

BUG-M033: Error response body not parsed
  Pattern: catch(e) { showError("Error") } — ignores server error message
  Fix: Parse error response body for specific error messages

BUG-M034: API response not validated
  Pattern: Assumes API always returns expected structure
  Fix: Validate response shape, handle missing/extra fields

BUG-M035: WebSocket not reconnecting
  Pattern: Connection drops, never reconnected
  Fix: Implement reconnection with exponential backoff
```

---

## 🔵 LOW SEVERITY BUGS (Code Quality / Best Practices)

```
BUG-L001: Magic numbers in code
  Pattern: if (status == 3) / SizedBox(height: 47.5)
  Fix: Use named constants: const STATUS_ACTIVE = 3

BUG-L002: Unused imports
  Pattern: Import statement for unused library
  Fix: Remove unused imports, configure linter

BUG-L003: Print statements in production code
  Pattern: print() / console.log() scattered throughout
  Fix: Use proper logging framework with log levels

BUG-L004: Inconsistent naming conventions
  Pattern: Mix of camelCase, snake_case, PascalCase for same type
  Fix: Follow language conventions, enforce with linter

BUG-L005: Dead code / unreachable code
  Pattern: Functions never called, conditions always true/false
  Fix: Remove dead code, it confuses maintainers

BUG-L006: TODO comments in production
  Pattern: // TODO: implement this / // FIXME: hack
  Fix: Create tickets for TODOs, remove or implement

BUG-L007: Missing documentation on public API
  Pattern: Public class/function without any documentation
  Fix: Add dartdoc/javadoc/docstring for public interfaces

BUG-L008: Overly long functions (>50 lines)
  Pattern: Single function doing 10 different things
  Fix: Extract into smaller, single-purpose functions

BUG-L009: Deep nesting (>4 levels)
  Pattern: if inside if inside if inside for inside try
  Fix: Extract conditions into functions, use early returns

BUG-L010: Hardcoded strings (not localized)
  Pattern: Text('Welcome back!') instead of Text(tr.welcomeBack)
  Fix: Extract to localization files

BUG-L011: Inconsistent error handling patterns
  Pattern: Some functions use try-catch, others use .catchError, others ignore errors
  Fix: Adopt one pattern consistently (prefer try-catch)

BUG-L012: Missing test coverage
  Pattern: Business logic without corresponding unit tests
  Fix: Write tests for critical paths: auth, payments, data processing

BUG-L013: Version not displayed in app
  Pattern: No way for user to tell which version they're running
  Fix: Show version in Settings/About screen

BUG-L014: No code formatting rules
  Pattern: Inconsistent formatting across files
  Fix: Configure formatter (dartfmt, prettier, ktfmt) and enforce in CI

BUG-L015: Deprecated API usage
  Pattern: Using APIs marked as @Deprecated
  Fix: Migrate to recommended replacement
```

---

## 📊 BUGS BY FRAMEWORK — QUICK REFERENCE

### Flutter-Specific Top 20
```
1.  StreamController not closed in dispose()
2.  setState called after dispose (mounted check missing)
3.  const constructor not used for static widgets
4.  BlocBuilder without buildWhen (rebuilds everything)
5.  Navigator.push without ever popping (stack grows)
6.  GlobalKey used unnecessarily (performance cost)
7.  MediaQuery.of(context) in build without caching
8.  Heavy computation in build() method
9.  Image.network without CachedNetworkImage
10. Missing ErrorWidget.builder for production
11. FutureBuilder without handling ConnectionState.waiting
12. Platform channel without PlatformException handling
13. Theme.of(context) deep in widget tree (could cache)
14. Scaffold.of(context) fails in same build context
15. GestureDetector absorbs taps meant for parent
16. Hero animation tag collision between screens
17. showDialog without WillPopScope/PopScope
18. TextEditingController.text read in build (not listener)
19. late variable crash on access before initialization
20. Riverpod ref.watch used in callbacks (should use ref.read)
```

### React Native-Specific Top 20
```
1.  useEffect without cleanup (memory leak)
2.  Inline style objects causing re-renders
3.  ScrollView wrapping FlatList (warning + performance)
4.  Missing key prop on list items
5.  Console.log left in production
6.  Async state update after unmount
7.  Bridge communication with large data (slow)
8.  Image component without cache configuration
9.  Native module missing on one platform
10. Hermes-specific behavior difference
11. React Navigation header not customized (default look)
12. AsyncStorage used for sensitive data
13. Missing ErrorBoundary for component crashes
14. Platform.OS check missing for platform-specific code
15. StatusBar style not matching screen theme
16. Keyboard avoiding view misconfigured
17. Deep linking not handling all route params
18. Push notification tap handler not set for killed state
19. Font scaling breaking layouts
20. TouchableOpacity inside ScrollView conflict
```

### Android Native (Kotlin) Top 20
```
1.  Activity leak from non-static inner class
2.  Room query without suspend (blocks main thread)
3.  Fragment transaction after onSaveInstanceState
4.  ViewBinding not cleared in onDestroyView
5.  LiveData observed with wrong lifecycle owner
6.  Missing INTERNET permission (obvious but happens)
7.  PendingIntent without FLAG_IMMUTABLE
8.  Exported component without intent filter security
9.  StrictMode violations (disk read on main thread)
10. RecyclerView without DiffUtil (full rebind on change)
11. CoroutineScope not cancelled in onDestroy
12. Handler post without removeCallbacks
13. SharedPreferences apply() vs commit() confusion
14. Content provider without permission enforcement
15. Notification channel not created (Android 8+)
16. Scoped storage not implemented (Android 10+)
17. WindowInsets not handled for edge-to-edge
18. WorkManager constraint misconfiguration
19. ProGuard rules missing for reflection-based libraries
20. BroadcastReceiver not unregistered in onStop
```

### Swift/iOS Top 20
```
1.  Strong reference cycle (missing [weak self] in closure)
2.  Force unwrap (!) on optional that can be nil
3.  NotificationCenter observer not removed
4.  CoreData context accessed from wrong thread
5.  UI update not dispatched to main queue
6.  Timer not invalidated on deinit
7.  UserDefaults used for sensitive data
8.  Missing NSCameraUsageDescription (crash or rejection)
9.  URLSession task not cancelled on dealloc
10. Delegate not declared as weak (retain cycle)
11. View controller presentation style incorrect
12. Safe area not respected (content under notch)
13. Info.plist missing required privacy descriptions
14. ATS exceptions too broad
15. Keychain access without error handling
16. Combine sink stored without cancellation
17. SwiftUI @State used for reference types (should be @StateObject)
18. Background task not ending properly
19. Deep link URL scheme conflict with other apps
20. Missing PrivacyInfo.xcprivacy (required 2024+)
```

---

## 🔍 HOW TO USE THIS DATABASE

During each review phase:
1. Identify the framework being used
2. Go to the relevant framework section above
3. Check EACH bug pattern against the actual codebase
4. For any match found, create a finding with proper citation
5. Reference this bug ID in your finding (e.g., "See BUG-C001 pattern")

This database is a LOOKUP TABLE, not a replacement for thorough analysis.
Real bugs are often COMBINATIONS of these patterns.
