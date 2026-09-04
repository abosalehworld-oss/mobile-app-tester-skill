# Phase 1: Architecture Review 🏗️

> **Objective:** Analyze the project's foundational structure, dependencies, design patterns,
> and code organization. Architectural problems are the ROOT CAUSE of most bugs — a bad
> architecture makes bugs inevitable.

---

## 🔍 PRE-CHECK: Project Discovery

Before analyzing architecture, gather this information:

### 1.1 Project Structure Map
```
ACTION: List the ENTIRE project directory tree (at least 3 levels deep)
OUTPUT: Complete tree showing all folders and files
LOOK FOR:
  - Is there a clear separation of concerns? (screens/views, models, services, etc.)
  - Are test directories present? (test/, integration_test/, etc.)
  - Is there a consistent naming convention?
  - Are there configuration files for different environments? (dev, staging, prod)
```

### 1.2 Dependency Audit
```
ACTION: Read the dependency file (pubspec.yaml / package.json / build.gradle / Podfile)
CHECK EACH DEPENDENCY FOR:
  ❑ Is it still maintained? (last update within 12 months)
  ❑ Are there known vulnerabilities? (check version numbers)
  ❑ Is the version pinned or floating? (^1.0.0 vs 1.0.0)
  ❑ Are there conflicting dependencies?
  ❑ Are dev dependencies properly separated from production dependencies?
  ❑ Is the minimum SDK version appropriate? (not too old, not too new)
  ❑ Are there unnecessary/unused dependencies?
```

### 1.3 Entry Point Analysis
```
ACTION: Read the main entry point file (main.dart / App.js / MainActivity.kt / AppDelegate.swift)
CHECK:
  ❑ Is initialization properly ordered? (services before UI)
  ❑ Are there blocking calls on the main thread during startup?
  ❑ Is error handling set up at the global level?
  ❑ Are environment configurations properly loaded?
  ❑ Is dependency injection / service locator configured?
```

---

## 📋 ARCHITECTURE CHECKS

### CHECK A1: Design Pattern Consistency
```
WHAT TO LOOK FOR:
  - Does the project follow a consistent architecture pattern?
    □ MVC (Model-View-Controller)
    □ MVVM (Model-View-ViewModel)
    □ MVP (Model-View-Presenter)
    □ Clean Architecture (Domain/Data/Presentation layers)
    □ BLoC/Cubit (Flutter-specific)
    □ Redux/MobX (React Native-specific)
    □ VIPER
    □ MVI (Model-View-Intent)

  - Is the pattern applied CONSISTENTLY across ALL features?
    (Common bug: Feature A uses MVVM, Feature B uses spaghetti code)

  - Are layer boundaries respected?
    (Common bug: UI layer directly calling database, skipping business logic)

CITATION REQUIRED: Show at least 2 examples of pattern usage (or violation)
```

### CHECK A2: Folder Structure Convention
```
EXPECTED STRUCTURES BY FRAMEWORK:

Flutter Clean Architecture:
  lib/
  ├── core/          (shared utilities, constants, themes)
  ├── features/      (feature-based modules)
  │   └── auth/
  │       ├── data/       (repositories, data sources, models)
  │       ├── domain/     (entities, use cases, repo interfaces)
  │       └── presentation/ (screens, widgets, blocs/cubits)
  ├── config/        (routes, themes, dependency injection)
  └── main.dart

React Native:
  src/
  ├── components/    (reusable UI components)
  ├── screens/       (screen-level components)
  ├── navigation/    (navigation configuration)
  ├── services/      (API calls, storage)
  ├── store/         (state management)
  ├── utils/         (helpers, constants)
  └── App.tsx

Native Android (Kotlin):
  app/src/main/
  ├── java/com/example/
  │   ├── data/      (repositories, database, network)
  │   ├── domain/    (models, use cases)
  │   ├── ui/        (activities, fragments, viewmodels)
  │   └── di/        (dependency injection modules)
  └── res/           (layouts, strings, drawables)

Native iOS (Swift):
  ProjectName/
  ├── Models/
  ├── Views/
  ├── ViewModels/
  ├── Services/
  ├── Extensions/
  ├── Resources/
  └── Supporting Files/

RED FLAGS:
  🔴 All files dumped in one folder
  🔴 No separation between UI and business logic
  🔴 Models defined inside UI files
  🔴 API calls made directly from UI components
  🔴 No dedicated folder for services/repositories
```

### CHECK A3: Dependency Injection
```
WHAT TO CHECK:
  ❑ Is there a DI framework? (get_it, provider, injectable / Dagger, Hilt / Swinject)
  ❑ Are dependencies injected or hardcoded?
  ❑ Are singletons used appropriately? (not overused)
  ❑ Can dependencies be mocked for testing?

COMMON BUGS:
  🐛 Service creates its own HTTP client instead of receiving one (untestable)
  🐛 Singleton database helper that can't be reset between tests
  🐛 Tight coupling between unrelated modules
  🐛 Circular dependencies between services
  🐛 Lazy singletons that throw on first access without error handling

CITATION REQUIRED: Show how dependencies are created and provided
```

### CHECK A4: Navigation Architecture
```
WHAT TO CHECK:
  ❑ Is navigation centralized? (route definitions in one place)
  ❑ Are routes type-safe? (no raw string routes like '/home')
  ❑ Is deep linking supported?
  ❑ Is the navigation stack managed properly? (no infinite stack growth)
  ❑ Are there route guards for authenticated screens?
  ❑ Is navigation state preserved on app restart?

COMMON BUGS:
  🐛 Navigator.push without ever popping (memory leak from infinite stack)
  🐛 Route names as magic strings scattered across codebase
  🐛 No redirect to login when auth token expires
  🐛 Deep link opens wrong screen or crashes
  🐛 Back button doesn't work correctly on Android
  🐛 Bottom navigation resets state on tab switch

CITATION REQUIRED: Show navigation setup and at least 2 navigation calls
```

### CHECK A5: Configuration & Environment Management
```
WHAT TO CHECK:
  ❑ Are there separate configs for dev/staging/production?
  ❑ Are API base URLs environment-specific?
  ❑ Are secrets properly handled? (NOT hardcoded, see Security Phase)
  ❑ Is the app build flavor/scheme properly configured?
  ❑ Are feature flags implemented?

RED FLAGS:
  🔴 Production API URL hardcoded in source code
  🔴 No environment switching mechanism
  🔴 Debug flags left enabled
  🔴 Console.log / print statements everywhere
  🔴 Same config used for all environments
```

### CHECK A6: Code Modularity & Coupling
```
WHAT TO CHECK:
  ❑ Can a feature be modified without touching other features?
  ❑ Are there shared components that could break multiple features if changed?
  ❑ Is the codebase monolithic or modular?
  ❑ Are there clear boundaries between modules?
  ❑ Is code duplication minimized?

MEASUREMENT:
  - Count imports: If a single file imports from 10+ different modules,
    it's a coupling red flag
  - Count file length: Files over 500 lines likely violate single responsibility
  - Count class methods: Classes with 20+ methods are doing too much

CITATION REQUIRED: Show the most coupled file (most imports) and explain why
```

### CHECK A7: Asset Management
```
WHAT TO CHECK:
  ❑ Are images optimized? (not raw 5MB photos in assets)
  ❑ Are assets organized in folders? (images/, icons/, fonts/)
  ❑ Are there unused assets? (bloating app size)
  ❑ Are resolution variants provided? (1x, 2x, 3x for iOS / mdpi-xxxhdpi for Android)
  ❑ Are fonts properly declared and loaded?
  ❑ Are localization files present and complete?

RED FLAGS:
  🔴 Massive image files (>1MB per image)
  🔴 Assets not declared in pubspec.yaml / asset catalogs
  🔴 Hardcoded strings instead of localization keys
  🔴 Missing dark mode assets
```

---

## 🚦 PHASE 1 GATE — MANDATORY CHECKLIST

You MUST complete ALL items before proceeding to Phase 2:

```
PHASE 1 GATE CHECKLIST:
  □ [A1] Design pattern identified and consistency verified
  □ [A2] Folder structure analyzed with RED FLAGS noted
  □ [A3] Dependency injection reviewed
  □ [A4] Navigation architecture mapped
  □ [A5] Configuration/environment management checked
  □ [A6] Code modularity and coupling assessed
  □ [A7] Asset management reviewed
  □ Project Structure Map produced
  □ Dependency Audit completed
  □ Entry Point Analysis completed
  □ Minimum 5 code citations provided
  □ Files examined list produced
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 1 COMPLETE: Architecture Review
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/12 items)

  🏗️ Architecture Health Score: [X/10]
  Pattern: [Detected Pattern]
  Modularity: [High/Medium/Low]
  Dependency Health: [Good/Warning/Critical]
═══════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**DO NOT proceed to Phase 2 until:**
1. All gate items are checked
2. Report is output to user
3. User acknowledges (or you've presented findings and are continuing the review)

---

## 🔗 WHAT FEEDS INTO LATER PHASES
- Architecture issues → Phase 3 (Logic bugs often stem from bad architecture)
- Navigation issues → Phase 2 (UI/UX depends on navigation)
- Dependency issues → Phase 4 (Vulnerable dependencies are security risks)
- Coupling issues → Phase 5 (Tight coupling causes performance problems)
