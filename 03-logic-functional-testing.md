# Phase 3: Logic & Functional Testing 🧠

> **Objective:** Trace every piece of business logic in the app. Verify that functions do
> what they claim to do, state transitions are correct, data flows properly, and edge cases
> are handled. Logic bugs are the HARDEST to find because the app "looks fine" but behaves wrong.

---

## 🎯 WHY THIS PHASE MATTERS

A button that navigates to the wrong screen is a UI bug. A button that charges the user
twice is a LOGIC bug. Logic bugs cause financial loss, data corruption, and user trust destruction.

---

## 📋 LOGIC & FUNCTIONAL CHECKS

### CHECK L1: Business Logic Correctness
```
ACTION: Identify ALL business rules in the app and verify their implementation

WHAT ARE BUSINESS RULES? Examples:
  - "User can only place an order if cart total > $10"
  - "Discount code can only be used once per user"
  - "User must verify email before accessing premium features"
  - "Maximum 5 items per order for free shipping"
  - "Session expires after 30 minutes of inactivity"

FOR EACH BUSINESS RULE:
  ❑ Is the rule implemented in code? (find the exact function)
  ❑ Is the rule enforced on BOTH client and server? (never trust client-only validation)
  ❑ Are boundary conditions correct? (>= vs >, <= vs <)
  ❑ Can the rule be bypassed? (by modifying request, skipping screen, etc.)
  ❑ Is the rule consistent across all entry points?

COMMON BUGS:
  🐛 Price calculation uses integer math instead of decimal (rounding errors)
  🐛 Discount applied multiple times (pressing button twice)
  🐛 Quantity check uses > instead of >= (off-by-one)
  🐛 Business rule only checked in UI but not in service layer
  🐛 Free trial check uses local device time (user can change clock)
  🐛 Age verification accepts future dates
  🐛 Currency conversion loses precision due to floating point

CITATION REQUIRED: List every business rule found and its implementation status
```

### CHECK L2: State Transition Verification
```
WHAT TO CHECK:
  Every feature has states. For example, an Order has:
  [Created] → [Confirmed] → [Processing] → [Shipped] → [Delivered]
                                                      → [Returned]
                          → [Cancelled]

  For EACH stateful entity in the app:
  ❑ Are all valid states defined? (enum or constants)
  ❑ Are transitions validated? (can't go from Delivered back to Created)
  ❑ Are invalid transitions handled gracefully?
  ❑ Is the current state displayed correctly in UI?
  ❑ Are state changes persisted? (survives app restart)
  ❑ Are concurrent state changes handled? (race conditions)

COMMON STATE BUGS:
  🐛 Order shows "Delivered" but user can still press "Cancel"
  🐛 Payment state gets stuck in "Processing" with no timeout
  🐛 User can submit form while state is already "submitting"
  🐛 App shows stale state after background refresh
  🐛 Two screens show different states for the same entity
  🐛 State machine allows impossible transitions (Cancelled → Shipped)

HOW TO VERIFY WITHOUT EMULATOR:
  1. Find all enum/constant definitions for states
  2. Find all state-changing functions
  3. Verify that each function checks current state before transitioning
  4. Look for missing transition guards

CITATION REQUIRED: Map at least one complete state machine with transitions
```

### CHECK L3: Data Transformation & Mapping
```
WHAT TO CHECK:
  Data flows through layers: API Response → DTO → Domain Model → UI Model

  ❑ Are all fields mapped correctly? (no missing fields)
  ❑ Are data types preserved? (String "123" → int 123, correct date parsing)
  ❑ Are null/empty values handled in transformations?
  ❑ Are units converted correctly? (API returns cents, UI shows dollars)
  ❑ Are date/time zones handled? (UTC ↔ local)
  ❑ Are enums mapped safely? (unknown enum value from API doesn't crash)

COMMON BUGS:
  🐛 API returns price in cents (1999), UI shows "1999" instead of "$19.99"
  🐛 Date string "2024-01-15" parsed with wrong format, shows wrong date
  🐛 API returns null for optional field → app crashes with NPE
  🐛 API adds new enum value → app crashes because switch has no default
  🐛 User timezone ignored → shows UTC times
  🐛 JSON key renamed on backend → field is always null on client
  🐛 Boolean field comes as String "true" → parsed as always-true object

VERIFICATION METHOD:
  1. Find all model/DTO classes
  2. Find all fromJson/toJson or equivalent mapping functions
  3. Check every field mapping for type safety
  4. Look for try-catch around parsing (or lack thereof)

CITATION REQUIRED: Show at least 3 data mapping functions and verify correctness
```

### CHECK L4: Conditional Logic & Branch Coverage
```
WHAT TO CHECK:
  ❑ Are all if/else branches reachable? (no dead code)
  ❑ Are nested conditions simplified? (not 5 levels of if/else)
  ❑ Are switch/when statements exhaustive? (handle all cases)
  ❑ Is the else/default case handled meaningfully? (not empty)
  ❑ Are boolean conditions correct? (&& vs ||, ! operator placement)
  ❑ Are ternary conditions readable and correct?

COMMON BUGS:
  🐛 if (user.isAdmin || user.isModerator && user.isVerified)
     → Missing parentheses! && binds tighter than ||
     → Correct: if (user.isAdmin || (user.isModerator && user.isVerified))

  🐛 if (list.length > 0) vs if (list.isNotEmpty)
     → Null-unsafe: list could be null

  🐛 if (status == 'active' || status == 'Active' || status == 'ACTIVE')
     → Should use case-insensitive comparison

  🐛 switch without default/else → crashes on unexpected value

  🐛 if (a != null) { use(a!) }
     → Race condition in multi-threaded: a could become null between check and use

  🐛 Nested ternary: x ? (y ? a : b) : (z ? c : d)
     → Unreadable, prone to logic errors

SEARCH PATTERNS:
  - Find all if/else chains longer than 3 branches
  - Find all switch/when statements, check for default
  - Find all ternary operators, check for nesting
  - Find all boolean expressions with mixed && and ||

CITATION REQUIRED: Show the most complex conditional logic and verify correctness
```

### CHECK L5: Loop & Iteration Safety
```
WHAT TO CHECK:
  ❑ Can any loop run infinitely? (while loops without break condition)
  ❑ Are loop boundaries correct? (off-by-one errors)
  ❑ Is the collection modified during iteration? (ConcurrentModificationException)
  ❑ Are nested loops necessary? (O(n²) performance risk)
  ❑ Are async operations in loops handled correctly?
  ❑ Is there proper error handling inside loops?

COMMON BUGS:
  🐛 while(true) with break condition that can never be reached
  🐛 for (i = 0; i <= list.length; i++) → IndexOutOfBounds (should be <)
  🐛 Removing items from list while iterating → crash or skipped items
  🐛 await inside for loop → serial execution, should use Future.wait/Promise.all
  🐛 Loop creates new objects without cleanup → memory leak
  🐛 Recursive function without base case → stack overflow

CITATION REQUIRED: Show every loop construct and verify termination conditions
```

### CHECK L6: Calculation & Arithmetic Accuracy
```
WHAT TO CHECK:
  ❑ Are monetary calculations using appropriate types? (Decimal, not double)
  ❑ Is integer overflow possible? (large quantities × prices)
  ❑ Is division by zero prevented?
  ❑ Are percentage calculations correct? (50% of 99 = 49.5, not 49)
  ❑ Are rounding rules consistent and correct?
  ❑ Are tax calculations using the right rates and order of operations?

COMMON BUGS:
  🐛 Total = price * quantity using double → $19.99 * 3 = $59.96999999
  🐛 Discount = total * discountPercent / 100 → integer division gives 0
  🐛 Average = sum / count where count could be 0
  🐛 Progress = completed / total → both ints, always 0 until complete
  🐛 Distance calculation with wrong formula or units
  🐛 Tax applied before discount instead of after (or vice versa)

VERIFICATION:
  1. Find all arithmetic operations
  2. Check data types involved
  3. Verify edge cases (0, negative, very large, very small)
  4. Check order of operations

CITATION REQUIRED: Show all monetary/critical calculations and verify correctness
```

### CHECK L7: Date & Time Logic
```
WHAT TO CHECK:
  ❑ Is timezone handling correct throughout?
  ❑ Are date comparisons correct? (same timezone?)
  ❑ Is daylight saving time (DST) handled?
  ❑ Are date parsing patterns correct and consistent?
  ❑ Are relative time displays correct? ("2 hours ago", "yesterday")
  ❑ Are date ranges validated? (start < end)
  ❑ Are leap years handled?
  ❑ Is the date library appropriate and consistent?

COMMON BUGS:
  🐛 DateTime.now() used everywhere instead of injected clock (untestable)
  🐛 Comparing dates in different timezones without normalization
  🐛 "Expires in 24 hours" but DST change makes it 23 or 25 hours
  🐛 Date format "dd/MM/yyyy" but user enters "MM/dd/yyyy"
  🐛 Age calculation wrong for Feb 29 birthdays
  🐛 Timer fires at wrong time after timezone change
  🐛 "Today" label wrong near midnight or timezone boundary
  🐛 Duration calculation ignores leap seconds / DST

CITATION REQUIRED: Show all date/time operations and verify timezone handling
```

### CHECK L8: Search, Filter & Sort Logic
```
WHAT TO CHECK:
  ❑ Does search handle empty query? (show all or show nothing?)
  ❑ Is search case-insensitive?
  ❑ Does search handle special characters? (accents, emoji, symbols)
  ❑ Is there debounce on search input? (not firing on every keystroke)
  ❑ Are filters applied cumulatively correctly?
  ❑ Does clearing one filter reset properly?
  ❑ Is sort order correct? (ascending/descending, locale-aware for strings)
  ❑ Are there results when combining conflicting filters?

COMMON BUGS:
  🐛 Search for "café" doesn't find "cafe" (accent handling)
  🐛 Sorting by name puts "Zebra" before "apple" (uppercase vs lowercase)
  🐛 Filter by date range includes boundary dates (or shouldn't)
  🐛 Search fires 100 API calls as user types 100 characters (no debounce)
  🐛 Removing filter doesn't clear the search results
  🐛 Sort indicator shows "ascending" but data is descending

CITATION REQUIRED: Show all search/filter/sort implementations
```

### CHECK L9: Authentication & Authorization Flow
```
WHAT TO CHECK:
  ❑ Can users access protected screens without authentication?
  ❑ Is token refresh handled correctly? (before it expires)
  ❑ Is logout clearing ALL local data? (token, cache, user preferences)
  ❑ Is "remember me" working correctly?
  ❑ Is biometric authentication implemented correctly?
  ❑ Is social login handling all error cases?
  ❑ Are role-based permissions enforced in UI?
  ❑ Is session timeout implemented?

COMMON BUGS:
  🐛 Token expires → API returns 401 → app shows generic error instead of login
  🐛 Logout clears token but cached data still shows on next login by different user
  🐛 Deep link to protected screen → crashes instead of redirect to login
  🐛 "Remember me" stores password in plain text
  🐛 Biometric prompt shown on devices without biometric hardware → crash
  🐛 Social login fails silently (no error message)
  🐛 Admin-only button visible to regular users (hidden but accessible)
  🐛 Refresh token logic creates infinite loop on expired refresh token

CITATION REQUIRED: Trace the complete auth flow from login to token usage to refresh to logout
```

### CHECK L10: Pagination & Infinite Scroll
```
WHAT TO CHECK:
  ❑ Does the first page load correctly?
  ❑ Does "load more" fetch the correct next page?
  ❑ Is there a loading indicator during page fetch?
  ❑ Is end-of-list handled? (no more data to load)
  ❑ Is duplicate data prevented? (same item appearing twice)
  ❑ Does pull-to-refresh reset pagination correctly?
  ❑ Is the page counter/cursor updated correctly?
  ❑ What happens on pagination error? (can the user retry?)

COMMON BUGS:
  🐛 Page 1 loads, user scrolls, page 2 loads, but page 1 data disappears
  🐛 Fast scroll → page 3, 4, 5 all fire simultaneously → duplicates
  🐛 Pull-to-refresh loads page 1 but page counter stays at 3
  🐛 Last page returns empty → infinite loading spinner
  🐛 Pagination uses offset instead of cursor → misses items added between pages
  🐛 Error on page 3 → no retry, user stuck with only 2 pages of data

CITATION REQUIRED: Show pagination implementation and verify correctness
```

---

## 🚦 PHASE 3 GATE — MANDATORY CHECKLIST

```
PHASE 3 GATE CHECKLIST:
  □ [L1]  Business rules identified and verified
  □ [L2]  State transitions mapped and validated
  □ [L3]  Data transformations checked for correctness
  □ [L4]  Conditional logic reviewed for correctness
  □ [L5]  Loops checked for safety and termination
  □ [L6]  Arithmetic/calculations verified (especially monetary)
  □ [L7]  Date/time logic verified (timezone, DST, formatting)
  □ [L8]  Search/filter/sort logic verified
  □ [L9]  Auth flow traced end-to-end
  □ [L10] Pagination verified for correctness
  □ Minimum 8 code citations provided
  □ Files examined list produced
  □ Cross-reference to Phase 1 architecture issues provided
```

### Gate Report Format:
```
══════════════════════════════════════════════════════════
  ✅ PHASE 3 COMPLETE: Logic & Functional Testing
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/13 items)

  🧠 Logic Health Score: [X/10]
  Business Rules Verified: [count]
  State Machines Mapped: [count]
  Calculations Verified: [count]
  Auth Flow: [Secure/Vulnerable/Not Found]
══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present all logic findings before proceeding to Phase 4 (Security).**
