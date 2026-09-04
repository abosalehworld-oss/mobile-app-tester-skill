# Phase 9: Error & Crash Prevention 🛡️

> **Objective:** Audit every error handling path in the app. Find unhandled exceptions,
> empty catch blocks, missing null checks, and crash-inducing edge cases. The goal is
> ZERO unhandled crashes reaching the user.

---

## 🎯 THE GOLDEN RULE OF ERROR HANDLING

> **Every error must either be HANDLED (recovered from) or REPORTED (to user + logging).**
> An error that is silently swallowed is WORSE than a crash — it causes invisible data
> corruption and impossible-to-debug issues.

---

## 📋 ERROR & CRASH PREVENTION CHECKS

### CHECK E1: Global Error Handling
```
WHAT TO CHECK:
  ❑ Is there a global error handler that catches unhandled exceptions?
  ❑ Does the global handler log the error? (for debugging)
  ❑ Does the global handler show user-friendly message? (not stack trace)
  ❑ Does the global handler report to crash analytics? (Crashlytics, Sentry)
  ❑ Does the app SURVIVE the error? (not crash to homescreen)

FRAMEWORK-SPECIFIC SETUP:

  Flutter:
  ❑ FlutterError.onError configured?
  ❑ PlatformDispatcher.instance.onError configured?
  ❑ runZonedGuarded wrapping runApp?
  ❑ ErrorWidget.builder customized for release?
  Example:
    void main() {
      runZonedGuarded(() {
        FlutterError.onError = (details) {
          FirebaseCrashlytics.instance.recordFlutterError(details);
        };
        runApp(MyApp());
      }, (error, stack) {
        FirebaseCrashlytics.instance.recordError(error, stack);
      });
    }

  React Native:
  ❑ ErrorBoundary component wrapping app?
  ❑ setJSExceptionHandler configured?
  ❑ setNativeExceptionHandler configured?
  ❑ Global Promise rejection handler?

  Android/Kotlin:
  ❑ Thread.setDefaultUncaughtExceptionHandler configured?
  ❑ CoroutineExceptionHandler for coroutines?
  ❑ ProcessLifecycleOwner for lifecycle-aware error handling?

  iOS/Swift:
  ❑ NSSetUncaughtExceptionHandler configured?
  ❑ Signal handlers for SIGABRT, SIGSEGV?

RED FLAGS:
  🔴 No global error handler at all → unhandled errors crash the app
  🔴 Global handler catches error but does nothing (empty handler)
  🔴 Global handler shows raw stack trace to user
  🔴 No crash reporting service configured (no Crashlytics/Sentry)
  🔴 Error handler only logs to console (lost in production)

CITATION REQUIRED: Show the global error handling setup (or note its absence)
```

### CHECK E2: Try-Catch Quality Audit
```
WHAT TO CHECK:
  Search the ENTIRE codebase for try-catch blocks and evaluate each:

  ❑ Is the catch block specific? (catches specific exception type, not generic)
  ❑ Is the caught error used? (not ignored)
  ❑ Is the error logged?
  ❑ Is there user feedback? (error message, retry option)
  ❑ Is there a finally block where needed? (cleanup resources)

ANTI-PATTERNS TO FIND:

  🔴 PATTERN 1: Empty Catch (Silent Failure)
  try {
    await fetchData();
  } catch (e) {
    // TODO: handle error    ← NEVER HANDLED
  }

  🔴 PATTERN 2: Catch-All Without Specificity
  try {
    await fetchData();
  } catch (e) {
    print(e);  ← Only prints, no user feedback, no recovery
  }

  🔴 PATTERN 3: Overly Broad Try Block
  try {
    // 50 lines of code with multiple possible errors
    // Impossible to know which line threw
  } catch (e) {
    showError("Something went wrong");  ← Useless message
  }

  🔴 PATTERN 4: Catch and Rethrow Without Context
  try {
    await fetchData();
  } catch (e) {
    throw e;  ← Loses stack trace in some languages
  }

  🔴 PATTERN 5: Catch Hiding Bugs
  try {
    int result = int.parse(userInput);
  } catch (e) {
    result = 0;  ← Silently uses wrong value instead of showing error
  }

CORRECT PATTERNS:
  ✅ Specific catch with user feedback:
  try {
    await apiService.createOrder(order);
  } on NetworkException catch (e) {
    showError("No internet. Please try again.");
    logger.warning("Order creation failed: network", error: e);
  } on ValidationException catch (e) {
    showFieldErrors(e.fieldErrors);
  } on ServerException catch (e) {
    showError("Server error. We're working on it.");
    crashlytics.recordError(e);
  }

SEARCH PATTERNS:
  - "catch" followed by empty body or just "print"
  - "catch (e) {}" or "catch (_) {}"
  - "// TODO" inside catch blocks
  - "catch (Exception" (too broad in many languages)

CITATION REQUIRED: Audit EVERY try-catch block, categorize as Good/Bad/Empty
```

### CHECK E3: Null Safety & Null Handling
```
WHAT TO CHECK:
  ❑ Is null safety enabled? (Dart sound null safety, Kotlin null safety)
  ❑ Are force-unwrap operators used safely? (!, !!, as)
  ❑ Are nullable types handled before use?
  ❑ Are API response fields treated as potentially null?
  ❑ Are map/list access operations null-safe?

DANGEROUS PATTERNS:

  Dart/Flutter:
  🔴 variable! (force unwrap without null check)
  🔴 as Type (force cast without is check)
  🔴 map['key'] used directly without null check
  🔴 list.first on potentially empty list
  🔴 Late variable accessed before initialization

  Kotlin:
  🔴 variable!! (non-null assertion)
  🔴 as Type (unsafe cast, use as? instead)
  🔴 lateinit accessed before init → UninitializedPropertyAccessException

  Swift:
  🔴 variable! (force unwrap)
  🔴 as! Type (force cast)
  🔴 try! (force try → crash on error)
  🔴 array[index] without bounds check

  React Native/JavaScript:
  🔴 data.user.name without optional chaining (data?.user?.name)
  🔴 array[0] on empty array
  🔴 JSON.parse without try-catch
  🔴 Object destructuring without defaults

SEARCH PATTERNS (HIGH PRIORITY):
  Dart: Search for "!" that isn't "!=" or "!is"
  Kotlin: Search for "!!" 
  Swift: Search for "!" in variable access (not "!=")
  JS/TS: Search for property access without "?" (optional chaining)

NULL CRASH SCENARIOS:
  🐛 API returns null for field that was always non-null → crash
  🐛 User deletes account → references to user object become null
  🐛 List empty → .first throws → crash
  🐛 Map key doesn't exist → null returned → used as non-null → crash
  🐛 Deep navigation argument is null → crash on screen open
  🐛 late variable accessed in dispose → crash (already cleaned up)

CITATION REQUIRED: Count ALL force-unwrap operators and assess each for safety
```

### CHECK E4: Edge Case & Boundary Testing
```
WHAT TO CHECK:
  For EVERY input/function, consider these edge cases:

  EMPTY/ZERO:
  ❑ Empty string input
  ❑ Empty list/array
  ❑ Zero quantity/amount
  ❑ Null/undefined input
  ❑ Empty search query
  ❑ Empty API response

  BOUNDARIES:
  ❑ Maximum input length
  ❑ Maximum integer value
  ❑ Minimum date/time
  ❑ Maximum list size
  ❑ First and last items in list
  ❑ Single item in list

  SPECIAL CHARACTERS:
  ❑ Unicode characters (emoji, CJK, Arabic, Hebrew)
  ❑ HTML entities (&amp;, &lt;, etc.)
  ❑ SQL special characters (', ", --)
  ❑ Newlines and tabs in text fields
  ❑ Very long strings (10,000+ characters)
  ❑ RTL text in LTR context (and vice versa)

  TIMING:
  ❑ Rapid consecutive taps
  ❑ Action during loading state
  ❑ Action during animation
  ❑ App just launched (state not ready)
  ❑ App returning from background
  ❑ Midnight/timezone boundary

  DEVICE CONDITIONS:
  ❑ Low memory warning
  ❑ Low storage
  ❑ No internet
  ❑ Slow internet (2G)
  ❑ Screen rotation during operation
  ❑ Interruption (phone call, notification)
  ❑ System kills app in background

CITATION REQUIRED: For each feature, identify at least 3 unhandled edge cases
```

### CHECK E5: Error Messages Quality
```
WHAT TO CHECK:
  ❑ Are error messages user-friendly? (not technical)
  ❑ Are error messages actionable? (tell user what to do)
  ❑ Are error messages specific? (not generic "error occurred")
  ❑ Are error messages localized?
  ❑ Do errors suggest a solution? (retry, check connection, contact support)

BAD ERROR MESSAGES:
  ❌ "Error"
  ❌ "Something went wrong"
  ❌ "null"
  ❌ "Exception: SocketException: Connection refused"
  ❌ "HTTP 500"
  ❌ "FormatException: Invalid date"
  ❌ "" (empty error message)

GOOD ERROR MESSAGES:
  ✅ "Unable to connect. Please check your internet and try again."
  ✅ "This email is already registered. Try logging in or use a different email."
  ✅ "Payment failed. Your card was not charged. Please try a different card."
  ✅ "Session expired. Please log in again to continue."
  ✅ "This feature requires camera access. Enable it in Settings."

ERROR MESSAGE CHECKLIST:
  □ Does it explain WHAT happened? (in plain language)
  □ Does it explain WHY? (if known)
  □ Does it suggest WHAT TO DO? (retry, change input, contact support)
  □ Does it avoid TECHNICAL JARGON? (no exception names, no stack traces)
  □ Does it avoid BLAMING THE USER? ("Invalid input" → "Please enter a valid email")

CITATION REQUIRED: List all error messages in the app and rate their quality
```

### CHECK E6: Crash Reporting & Monitoring
```
WHAT TO CHECK:
  ❑ Is a crash reporting tool integrated? (Crashlytics, Sentry, Bugsnag)
  ❑ Are non-fatal errors also reported? (not just crashes)
  ❑ Is user context attached to crash reports? (user ID, screen, action)
  ❑ Are breadcrumbs logged? (sequence of actions before crash)
  ❑ Are debug symbols / source maps uploaded for readable stack traces?
  ❑ Is crash reporting NOT sending PII? (privacy compliance)
  ❑ Are custom keys set for easier debugging?

PROPER SETUP:
  ✅ Crashlytics/Sentry initialized early in app startup
  ✅ User ID set after login (for tracking affected users)
  ✅ Custom keys: current_screen, last_api_call, app_version
  ✅ Breadcrumbs: user actions logged in sequence
  ✅ Source maps/dSYMs uploaded for each release
  ✅ Alerts configured for crash rate spikes

RED FLAGS:
  🔴 No crash reporting at all → blind to production crashes
  🔴 Crash reporting initialized AFTER other code → misses startup crashes
  🔴 No source maps → unreadable minified stack traces
  🔴 PII in crash reports → privacy violation
  🔴 Crash rate not monitored → problems discovered from bad reviews

CITATION REQUIRED: Show crash reporting setup and configuration
```

### CHECK E7: Loading & Empty States
```
WHAT TO CHECK:
  Every screen/component that loads data must handle ALL states:

  ❑ LOADING: Is there a clear loading indicator?
  ❑ SUCCESS: Is data displayed correctly?
  ❑ ERROR: Is there an error message with retry option?
  ❑ EMPTY: Is there a meaningful empty state? (not just blank screen)
  ❑ REFRESHING: Is pull-to-refresh or refresh button available?

STATE MACHINE FOR DATA LOADING:
  [Initial] → [Loading] → [Success with Data]
                        → [Success but Empty]
                        → [Error]
  [Any State] → [Refreshing] → [Success/Error]

COMMON BUGS:
  🐛 No loading indicator → user thinks app is frozen
  🐛 Loading spinner never stops on error (infinite loading)
  🐛 Empty list shows completely blank screen (user confused)
  🐛 Error state has no retry button (user must restart app)
  🐛 Loading indicator covers interaction elements (can't cancel)
  🐛 Multiple loading indicators on same screen
  🐛 Skeleton loading doesn't match actual layout
  🐛 Loading state persists after screen returns from background

CITATION REQUIRED: For each data-loading screen, verify all 4 states are handled
```

### CHECK E8: Graceful Degradation
```
WHAT TO CHECK:
  ❑ If a feature fails, does the rest of the app still work?
  ❑ If analytics SDK crashes, does the app survive?
  ❑ If push notification fails, can the user still use the app?
  ❑ If one API call fails, do other features still work?
  ❑ If a third-party SDK has a breaking update, is it isolated?

ISOLATION PRINCIPLE:
  ✅ Non-critical features wrapped in try-catch (ads, analytics, reviews)
  ✅ Third-party SDK calls behind abstraction layer
  ✅ Feature flags to disable broken features remotely
  ✅ Fallback UI when component crashes (ErrorBoundary/ErrorWidget)

RED FLAGS:
  🔴 Analytics crash takes down entire app
  🔴 Ad SDK error causes white screen
  🔴 Feature flag service unavailable → all features disabled
  🔴 One broken image crashes entire image gallery
  🔴 Corrupt cache crashes app on startup (can't even clear cache)

CITATION REQUIRED: Show isolation/protection for at least 3 non-critical features
```

---

## 🚦 PHASE 9 GATE — MANDATORY CHECKLIST

```
PHASE 9 GATE CHECKLIST:
  □ [E1] Global error handling verified
  □ [E2] Try-catch quality audit completed
  □ [E3] Null safety audit completed
  □ [E4] Edge cases identified for major features
  □ [E5] Error messages quality assessed
  □ [E6] Crash reporting setup verified
  □ [E7] Loading/empty/error states verified per screen
  □ [E8] Graceful degradation checked
  □ Minimum 6 code citations provided
  □ Files examined list produced
  □ Cross-reference to Phase 4 security error handling
```

### Error Handling Heatmap (MANDATORY):
```
For each screen/service, rate error handling:
┌──────────────────────┬─────────┬───────────┬───────────┐
│ Screen/Service       │ Try-    │ Null      │ Loading   │
│                      │ Catch   │ Safety    │ States    │
├──────────────────────┼─────────┼───────────┼───────────┤
│ Login Screen         │ ✅/⚠️/❌ │ ✅/⚠️/❌   │ ✅/⚠️/❌   │
│ Home Screen          │ ✅/⚠️/❌ │ ✅/⚠️/❌   │ ✅/⚠️/❌   │
│ Profile Service      │ ✅/⚠️/❌ │ ✅/⚠️/❌   │ N/A       │
│ ...                  │ ...     │ ...       │ ...       │
└──────────────────────┴─────────┴───────────┴───────────┘
```

### Gate Report Format:
```
══════════════════════════════════════════════════════════
  ✅ PHASE 9 COMPLETE: Error & Crash Prevention
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/11 items)

  🛡️ Error Handling Score: [X/10]
  Empty Catch Blocks: [count]
  Force Unwraps: [count]
  Missing Loading States: [count]
  Crash Reporting: [Configured/Missing]
  Error Message Quality: [Good/Fair/Poor]
══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present error handling findings before proceeding to Phase 10 (Final Delivery).**
