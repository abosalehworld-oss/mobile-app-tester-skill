# Changelog

All notable changes to the Mobile App Tester Skill are documented here.

## [2.0.0] - 2026-09-06

### Added
- **Phase 13: Pre-Delivery Sentry Validation** (`13-pre-delivery-sentry-validation.md`)
  - Step-by-step Sentry setup guide for all mobile frameworks
  - Real-device testing scenarios (normal flow, edge cases, attacks)
  - Final deployment verdict with Sentry data
- **Rule 10: MANDATORY FRESH-EYES RE-ANALYSIS** in `SKILL.md`
  - Second independent pass to catch missed vulnerabilities
  - 4-layer reminder system (SKILL.md rule + Phase 9 reminder + Phase 10 top mega-reminder + Phase 10 bottom execution instructions)
  - Proven to catch Critical bugs missed on first pass
- **Rule 11: MANDATORY WEB SEARCH** for current-year vulnerabilities in `SKILL.md`
  - AI must search web for current OWASP Mobile vulnerabilities before Phase 4
  - Must include source URLs in report
- **Hacker Mindset R6** in `11-remediation-execution.md`
  - Post-fix verification from attacker's perspective

### Changed
- Citation minimums raised from 65 to **91** across all phases
- Phase gates raised from 12 to **13**
- `04-security-audit.md` — Added mandatory web search gate at top
- `09-error-crash-prevention.md` — Added Fresh-Eyes pre-flight reminder
- `10-final-delivery-checklist.md` — Added 🔴🔴🔴 mega-reminder + full Fresh-Eyes instructions
- Cross-Reference Matrix updated to require 91 minimum citations

## [1.2.0] - 2026-09-05

### Added
- **Phase 12: Store & Commercial Readiness** (`12-store-commercial-readiness.md`)
  - Full Google Play + Apple App Store submission readiness verification
  - Payment gateway security audit (Stripe, PayPal, Google Play Billing, StoreKit)
  - Revenue/monetization system verification (ads, subscriptions, in-app purchases)
  - Revenue protection checks (server-side validation, tamper-proofing)
  - Mandatory web search for current store policies (not relying on training data)
  - Store readiness matrix with YES/NO commercial release verdict
- **Rule 8: Language Adaptation** in `SKILL.md`
  - AI must respond in the same language the user writes
- **Rule 9: Web Search for Current Policies** in `SKILL.md`
  - AI must search the web for current store policies in Phase 12
  - Must cite source URLs for every policy requirement referenced

### Changed
- Phase overview table updated to include Phase 12
- Phase file references updated to include Phase 12

## [1.1.0] - 2026-09-05

### Added
- **Phase 11: Structured Remediation** (`11-remediation-execution.md`)
  - Sprint-based fix execution (BLOCKER → HIGH → MEDIUM → LOW)
  - Mandatory before/after diff for every fix
  - Per-sprint verification gates with user confirmation
  - Anti-scope-creep rules (fix ONLY reported findings)
  - Final remediation summary report
- **Rule 7: NO AUTO-FIX** in `SKILL.md`
  - Phases 1-10 are now explicitly READ-ONLY analysis
  - AI is forbidden from modifying source code during analysis
  - Users directed to Phase 11 for structured fixes
- **Formatting Rules** in `10-final-delivery-checklist.md`
  - Format Rule 1: No ASCII art boxes (use Markdown tables)
  - Format Rule 2: Clear section separation with `---` and headers
  - Format Rule 3: Final report must be saved as artifact file
  - Format Rule 4: Full Findings Registry (F4) is mandatory

### Changed
- All ASCII art templates in Phase 10 replaced with standard Markdown tables
- Executive Summary template now uses `| Field | Value |` format
- Cross-Reference Matrix uses standard Markdown table
- Critical Path table uses standard Markdown table
- Phase Health Scores uses standard Markdown table
- Final Sign-Off uses standard Markdown table with Phase 11 redirect
- `11-common-bugs-database.md` renamed to `ref-common-bugs-database.md` (no longer conflicts with Phase 11 numbering)
- Phase overview table in SKILL.md updated to include Phase 11 (optional)
- Phase file references in SKILL.md updated

## [1.0.0] - 2026-09-04

### Added
- Initial release of the comprehensive mobile app testing skill
- 10 structured review phases with mandatory gates and STOP points
- Anti-laziness enforcement system with 4 built-in mechanisms:
  - Proof-of-work citations (65 minimum total across all phases)
  - Mandatory phase gate checklists
  - Mandatory STOP points between phases
  - Cross-reference verification matrix in final phase
- Full OWASP Mobile Top 10 (2024) security coverage
- Support for 9 mobile frameworks:
  - Flutter / Dart
  - React Native (JavaScript / TypeScript)
  - Kotlin (Android Native)
  - Swift (iOS Native)
  - Jetpack Compose
  - SwiftUI
  - Xamarin / MAUI
  - Ionic / Capacitor
  - Kotlin Multiplatform (KMP)
- 200+ real-world bug patterns in `11-common-bugs-database.md`
- Per-framework top-20 bug lists
- Platform compliance matrices (Android / iOS)
- Store submission readiness checklists (Google Play + Apple App Store)
- Executive summary with health score (0-100) and release recommendation
- Severity classification system (Critical / High / Medium / Low)
- Framework auto-detection from project files
- Bilingual documentation (English + Arabic)
