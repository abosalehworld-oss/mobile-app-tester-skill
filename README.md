# Mobile App Comprehensive Tester Skill 📱

**The most thorough AI-powered mobile application testing skill on the market.**  
Transform any AI agent into a Senior Mobile QA Engineer through 10 structured review phases, mandatory gates, and an anti-laziness enforcement system that guarantees real code citations for every finding.

[![skills.sh](https://img.shields.io/badge/skills.sh-compatible-brightgreen)](https://skills.sh)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Frameworks](https://img.shields.io/badge/Frameworks-Flutter%20%7C%20React%20Native%20%7C%20Kotlin%20%7C%20Swift-blue)](SKILL.md)

---

## ✨ What This Skill Does

When installed, your AI agent becomes a **professional mobile app QA engineer** capable of:

- 🏗️ **Architecture Review** — Design patterns, dependencies, folder structure, coupling
- 🎨 **UI/UX Testing** — Widgets, forms, buttons, responsive design, accessibility, dark mode
- 🧠 **Logic & Functional Testing** — Business rules, state machines, calculations, auth flows
- 🔒 **Security Audit** — Full OWASP Mobile Top 10 coverage, zero compromise on safety
- ⚡ **Performance Analysis** — Memory leaks, main thread violations, startup time, battery drain
- 📱 **Platform Compatibility** — Android/iOS-specific checks, permissions, deep links, notifications
- 🌐 **API & Network Resilience** — Error handling for all HTTP status codes, offline mode, caching
- 🔄 **State & Data Management** — Race conditions, data consistency, local database integrity
- 🛡️ **Error & Crash Prevention** — Null safety, global error handlers, edge cases, loading states
- ✅ **Final Delivery** — Executive summary, cross-reference proof matrix, store submission checklist

---

## 🚀 Install in 30 Seconds

```bash
npx skills add https://github.com/YOUR_USERNAME/mobile-app-tester-skill
```

Or globally:

```bash
npx skills add https://github.com/YOUR_USERNAME/mobile-app-tester-skill -g -y
```

---

## 🎯 Supported Frameworks

| Framework | Detection |
|-----------|-----------|
| **Flutter / Dart** | `pubspec.yaml`, `.dart` files |
| **React Native** | `package.json` + react-native, `.tsx/.jsx` |
| **Kotlin (Android)** | `build.gradle.kts`, `.kt`, `AndroidManifest.xml` |
| **Swift (iOS)** | `*.xcodeproj`, `.swift`, `Info.plist` |
| **Jetpack Compose** | `.kt` with `@Composable` |
| **SwiftUI** | `.swift` with `struct X: View` |
| **Xamarin / MAUI** | `*.csproj`, `.xaml`, `MauiProgram.cs` |
| **Ionic / Capacitor** | `ionic.config.json`, `capacitor.config.ts` |
| **KMP** | `shared/`, `composeApp/`, `iosApp/` |

---

## 🛡️ Anti-Laziness Enforcement System

This skill is engineered to **force thorough review** even from AI agents that tend to skip steps.  
Four built-in mechanisms make skipping impossible:

### 1. Proof-of-Work Citations (65 Minimum)
Every finding MUST include exact file path, line numbers, and copied code snippet.  
`❌ FORBIDDEN: "I reviewed auth and found no issues"`  
`✅ REQUIRED: "In lib/services/auth_service.dart:45-52, token stored unencrypted..."`

### 2. Mandatory Phase Gates (9 Gates)
Each of the 10 phases has a structured checklist that must be completed with evidence before the AI can proceed to the next phase.

### 3. Mandatory STOP Points
After each phase, the AI MUST output a structured report and wait for acknowledgment before continuing.

### 4. Cross-Reference Verification Matrix
The final phase requires a matrix proving every file was examined with citation counts — if any phase shows 0 files, the entire review is declared invalid.

---

## 📂 Skill Structure

```
mobile-app-tester-skill/
├── SKILL.md                        ← Main entry + enforcement rules
├── 01-architecture-review.md       ← Phase 1: Project structure & dependencies
├── 02-ui-ux-testing.md             ← Phase 2: Screens, widgets, accessibility
├── 03-logic-functional-testing.md  ← Phase 3: Business logic & state transitions
├── 04-security-audit.md            ← Phase 4: OWASP Mobile Top 10
├── 05-performance-optimization.md  ← Phase 5: Memory, CPU, battery, startup
├── 06-platform-compatibility.md    ← Phase 6: Android/iOS platform specifics
├── 07-api-network-resilience.md    ← Phase 7: HTTP, offline, caching, retry
├── 08-state-data-management.md     ← Phase 8: State architecture & data flow
├── 09-error-crash-prevention.md    ← Phase 9: Null safety & crash prevention
├── 10-final-delivery-checklist.md  ← Phase 10: Executive summary & sign-off
└── 11-common-bugs-database.md      ← 200+ real-world bug patterns reference
```

---

## 💬 How to Use with Your AI Agent

### Start a Full Review
```
Review my Flutter project using the mobile-app-tester skill.
Start from Phase 1 and follow ALL mandatory gates. Do not skip phases.
My project is at: [path to your project]
```

### Security-Only Review
```
Using the mobile-app-tester skill, run ONLY Phase 4 (Security Audit)
on my React Native project. Follow all OWASP Mobile Top 10 checks.
```

### Quick UI Check
```
Run Phase 2 (UI/UX Testing) from the mobile-app-tester skill on my
screens folder. Check all buttons, forms, and responsive design.
```

### Pre-Release Final Check
```
My app is ready for release. Run Phase 10 (Final Delivery Checklist)
from the mobile-app-tester skill and give me a release recommendation.
```

---

## 📊 What You Get from Each Review

### Phase Report (After Each Phase)
```
═══════════════════════════════════════════════════════
  ✅ PHASE 4 COMPLETE: Security Audit
  📊 Findings: 2 Critical | 3 High | 5 Medium | 2 Low
  📋 Gate Status: PASSED (14/14 items)

  🔒 Security Health Score: 4/10
  OWASP Compliance: 6/10 categories passed
  Hardcoded Secrets: 2 found
  Insecure Storage: 3 found
═══════════════════════════════════════════════════════
```

### Final Executive Summary
```
╔═════════════════════════════════════════╗
║         📱 MOBILE APP QA REPORT         ║
║         Overall Score: 61/100           ║
║  🔴 Critical: 4  🟠 High: 7            ║
║  🟡 Medium: 12   🔵 Low: 8             ║
║  Release Recommendation: 🟡 WITH FIXES  ║
╚═════════════════════════════════════════╝
```

---

## 🔍 Sample Finding Format

```markdown
### 🔴 [F001]: Auth token stored unencrypted

**Location:** `lib/services/auth_service.dart:67-71`
**Category:** Security
**Impact:** Token accessible to any root app — full session hijack possible

**Problematic Code:**
```dart
// Lines 67-71
final prefs = await SharedPreferences.getInstance();
prefs.setString('auth_token', response.token);
prefs.setString('refresh_token', response.refreshToken);
```

**Why This Is A Problem:**
SharedPreferences stores data as plain text XML on Android. Any app with
root access, or any backup extraction, reveals the token completely.

**Fix:**
```dart
final storage = FlutterSecureStorage();
await storage.write(key: 'auth_token', value: response.token);
```
**References:** OWASP M9 - Insecure Data Storage
```

---

## 🤝 Contributing

Found a common bug pattern not in the database? Have a framework-specific check to add?  
PRs are welcome! Please follow the existing format in `11-common-bugs-database.md`.

---

## 📜 License

MIT — free to use, share, and modify.

---

## 🌟 Activate on skills.sh

```bash
npx skills add https://github.com/YOUR_USERNAME/mobile-app-tester-skill -g -y
```

> Built with ❤️ for mobile developers who can't always run an emulator.

---

## 👤 Author

**Mohamed Saleh** 🇪🇬  
Egypt — Mobile & Software Developer  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mohamed%20Saleh-blue?logo=linkedin)](https://www.linkedin.com/in/mr-mohamed-saleh/)
