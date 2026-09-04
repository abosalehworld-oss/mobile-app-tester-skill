# Phase 8: State & Data Management 🔄

> **Objective:** Audit the app's state management architecture for correctness, consistency,
> memory safety, and data integrity. State bugs are among the most confusing — the app
> shows wrong data, stale data, or different data on different screens for the same entity.

---

## 🎯 WHY THIS PHASE MATTERS

State management is the NERVOUS SYSTEM of a mobile app. When state is wrong:
- User sees outdated prices and orders wrong item
- User edits profile but old data persists
- Two screens show different information for the same entity
- App appears "frozen" because state update never triggered UI refresh

---

## 📋 STATE & DATA MANAGEMENT CHECKS

### CHECK D1: State Architecture Assessment
```
WHAT TO CHECK:
  ❑ What state management solution is used?

  FLUTTER:
    □ setState (built-in)
    □ Provider / Riverpod
    □ BLoC / Cubit
    □ GetX
    □ MobX
    □ Redux
    □ ValueNotifier / ChangeNotifier
    □ Signals

  REACT NATIVE:
    □ useState / useReducer (built-in)
    □ Redux / Redux Toolkit
    □ MobX
    □ Zustand
    □ Jotai / Recoil
    □ Context API
    □ React Query / TanStack Query

  KOTLIN/ANDROID:
    □ ViewModel + LiveData
    □ ViewModel + StateFlow
    □ ViewModel + Compose State
    □ MVI pattern

  SWIFT/IOS:
    □ @State / @StateObject / @ObservedObject
    □ Combine
    □ The Composable Architecture (TCA)
    □ MVVM with ObservableObject

  FOR THE CHOSEN SOLUTION:
  ❑ Is it used CONSISTENTLY across the entire app?
  ❑ Is the global vs local state separation clear?
  ❑ Is state scoped correctly? (not everything global)
  ❑ Is there a single source of truth for each piece of data?

COMMON MISTAKES:
  🐛 Mixing 3+ state management solutions (chaos)
  🐛 Everything stored in global state (even screen-local state)
  🐛 Same data duplicated in multiple state containers
  🐛 State management chosen but not understood (misused patterns)
  🐛 No state management at all (everything in widget/component state)

CITATION REQUIRED: Identify the state management solution and verify consistency
```

### CHECK D2: State Lifecycle & Cleanup
```
WHAT TO CHECK:
  ❑ Are state containers created at the right scope?
  ❑ Are state containers disposed/cleaned when no longer needed?
  ❑ Are subscriptions to state unsubscribed on dispose?
  ❑ Is state reset when user logs out?
  ❑ Is state preserved across configuration changes? (rotation)
  ❑ Is state restored after process death? (Android background kill)

FLUTTER-SPECIFIC:
  ❑ BLoC/Cubit closed in dispose()?
  ❑ StreamSubscription cancelled?
  ❑ Provider scoped to route (not app-level for screen-level state)?
  ❑ ChangeNotifier disposed?
  ❑ Riverpod autoDispose used for temporary state?

REACT NATIVE-SPECIFIC:
  ❑ Redux store properly configured with middleware?
  ❑ useEffect cleanup functions returning unsubscribe?
  ❑ Context providers not wrapping entire app unnecessarily?
  ❑ React Query cache properly configured with stale time?

ANDROID-SPECIFIC:
  ❑ ViewModel scoped to correct lifecycle owner?
  ❑ SavedStateHandle used for process death survival?
  ❑ Flow collection uses repeatOnLifecycle?
  ❑ LiveData observers removed? (auto with lifecycle)

iOS-SPECIFIC:
  ❑ Combine cancellables stored and cancelled?
  ❑ ObservableObject properly scoped?
  ❑ @StateObject vs @ObservedObject used correctly?
  ❑ Scene phase changes handled?

COMMON LEAKS:
  🐛 BLoC created but never closed → stream stays open forever
  🐛 Provider at app level for screen-local data → never garbage collected
  🐛 Redux store accumulates data from every screen visit
  🐛 React Query cache grows unbounded without gcTime
  🐛 ViewModel retained after Fragment destroyed (navigation leak)

CITATION REQUIRED: Show state lifecycle management for at least 3 state containers
```

### CHECK D3: Data Consistency & Single Source of Truth
```
WHAT TO CHECK:
  ❑ Is each piece of data stored in exactly ONE place?
  ❑ When data is updated, do ALL references update?
  ❑ Are there multiple copies of the same data that could diverge?
  ❑ Is optimistic update implemented correctly? (rollback on failure)
  ❑ Is there a synchronization mechanism for related data?

SCENARIOS TO VERIFY:
  1. User edits profile on Screen A → navigates to Screen B
     → Does Screen B show the updated profile?

  2. User adds item to cart on Screen A → switches to Cart tab
     → Does cart show the new item?

  3. User receives push notification about order status change
     → Does the order detail screen update without manual refresh?

  4. Two users edit the same resource simultaneously
     → Is the conflict handled? (last-write-wins, merge, or error?)

COMMON BUGS:
  🐛 Profile screen fetches its own copy of user data → stale after edit
  🐛 Cart count in tab badge ≠ actual cart items count
  🐛 Deleting item from list → item still visible in detail screen
  🐛 Optimistic update succeeds visually but server fails → inconsistency
  🐛 Cache shows old data after successful server update
  🐛 Parent screen doesn't refresh after child screen changes data

DATA FLOW PATTERN TO VERIFY:
  API → Repository → State Container → UI
  (Update should flow the same path in reverse)
  UI → State Container → Repository → API → Repository → State → UI (confirm)

CITATION REQUIRED: Trace data update flow for at least 2 user actions
```

### CHECK D4: Race Conditions & Concurrency
```
WHAT TO CHECK:
  ❑ Can two async operations update the same state simultaneously?
  ❑ Is debouncing used for rapid user inputs? (search, button taps)
  ❑ Are API responses handled in order? (request A then B, but B returns first)
  ❑ Is there mutex/lock for shared resources?
  ❑ Can the user trigger an action while a previous action is still processing?

COMMON RACE CONDITIONS:
  🐛 Search: Type "abc" → request for "abc" starts → type "abcd" → request for "abcd" starts
     → "abc" response arrives AFTER "abcd" → shows wrong results

  🐛 Double tap: User taps "Buy" twice quickly → two orders created

  🐛 Navigate: User opens screen → data loading → user goes back → user opens again
     → two loading operations running → response from first visit shown on second

  🐛 Refresh: Pull-to-refresh while auto-refresh timer fires → duplicate data

  🐛 Logout race: User presses logout → logout API call starts → background refresh
     fires → new token requested → user is logged back in

SOLUTIONS TO LOOK FOR:
  ✅ Debounce on search input (300-500ms)
  ✅ Loading flag that prevents duplicate submissions
  ✅ Request cancellation on new request
  ✅ Sequence number/timestamp on responses to discard stale
  ✅ Mutex/lock for database write operations
  ✅ Event queue for ordered processing

CITATION REQUIRED: Show at least 2 areas susceptible to race conditions and their mitigation
```

### CHECK D5: Local Database & Persistence
```
WHAT TO CHECK:
  ❑ What local database is used? (SQLite, Realm, Hive, Room, CoreData, MMKV)
  ❑ Is the database schema versioned?
  ❑ Are migrations defined for schema changes?
  ❑ Is there data validation before writing?
  ❑ Are database operations transactional where needed?
  ❑ Is the database encrypted for sensitive data?
  ❑ Is database access thread-safe?
  ❑ Is old data cleaned up periodically?

COMMON BUGS:
  🐛 Migration missing → app crashes on update for existing users
  🐛 Migration drops table → user loses all local data
  🐛 No transaction for related writes → partial data on crash
  🐛 Database write on main thread → UI freezes
  🐛 No unique constraints → duplicate entries
  🐛 Database file grows forever → fills storage
  🐛 Database not closed on app exit → corruption risk
  🐛 Different database versions between fresh install and upgrade

MIGRATION SAFETY CHECKLIST:
  □ Is every schema change covered by a migration?
  □ Do migrations preserve existing data?
  □ Is migration tested from every previous version?
  □ Is there a fallback if migration fails?
  □ Are destructive migrations explicitly handled?

CITATION REQUIRED: Show database setup, schema, and migration handling
```

### CHECK D6: Shared Preferences / Key-Value Storage
```
WHAT TO CHECK:
  ❑ What is stored in key-value storage? (list all keys)
  ❑ Is sensitive data stored here? (SHOULD NOT BE — use secure storage)
  ❑ Are keys defined as constants? (not magic strings)
  ❑ Are default values provided for missing keys?
  ❑ Is type safety enforced? (getString returns String, not dynamic)
  ❑ Is data cleared on logout?
  ❑ Is data size reasonable? (not storing large objects)

RED FLAGS:
  🔴 SharedPreferences.setString('token', authToken) → INSECURE
  🔴 SharedPreferences.setString('user', jsonEncode(hugeUserObject)) → TOO LARGE
  🔴 Key spelled differently in read vs write → always returns null
  🔴 No default value → null crash
  🔴 Data persists after logout → next user sees previous user's data
  🔴 Synchronous read on app startup → blocks main thread

PROPER USAGE:
  ✅ Theme preference (light/dark)
  ✅ Language preference
  ✅ "Has seen onboarding" flag
  ✅ Last selected tab
  ✅ Non-sensitive user settings
  ❌ Auth tokens, passwords, PII, large datasets

CITATION REQUIRED: List ALL key-value storage keys and their sensitivity level
```

### CHECK D7: Global State Pollution
```
WHAT TO CHECK:
  ❑ Are there global variables that any part of the app can modify?
  ❑ Is singleton state mutable from outside its intended scope?
  ❑ Are there static mutable variables?
  ❑ Can a bug in one feature corrupt state used by another feature?

RED FLAGS:
  🔴 Global mutable map: Map<String, dynamic> appState = {};
  🔴 Static mutable field: static var currentUser = null;
  🔴 Singleton with public setter: AppConfig.instance.apiUrl = "...";
  🔴 Service locator returning mutable instances that are shared

PROPER PATTERNS:
  ✅ Immutable state objects (copyWith pattern)
  ✅ State only modified through defined actions/events
  ✅ Unidirectional data flow (state → UI, events → state)
  ✅ Encapsulated state with public getters, private setters

CITATION REQUIRED: Search for global mutable state and assess risk
```

---

## 🚦 PHASE 8 GATE — MANDATORY CHECKLIST

```
PHASE 8 GATE CHECKLIST:
  □ [D1] State architecture identified and consistency verified
  □ [D2] State lifecycle and cleanup audited
  □ [D3] Data consistency and single source of truth verified
  □ [D4] Race conditions and concurrency assessed
  □ [D5] Local database and migrations checked
  □ [D6] Key-value storage audited
  □ [D7] Global state pollution checked
  □ Minimum 5 code citations provided
  □ Files examined list produced
  □ State flow diagram produced
```

### State Flow Diagram (MANDATORY):
```
Produce a text-based diagram showing:
  User Action → Event/Action → State Container → New State → UI Update

Example:
  [Tap Buy Button]
    → CartBloc.add(AddToCart(product))
      → CartState(items: [...old, product], loading: false)
        → CartScreen rebuilds with new item
          → Tab badge updates count
```

### Gate Report Format:
```
══════════════════════════════════════════════════════════
  ✅ PHASE 8 COMPLETE: State & Data Management
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/10 items)

  🔄 State Health Score: [X/10]
  State Solution: [identified solution]
  Consistency: [Single Source/Duplicated/Fragmented]
  Race Conditions: [count found]
  Data Persistence: [Secure/Insecure/Mixed]
══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present state management findings before proceeding to Phase 9.**
