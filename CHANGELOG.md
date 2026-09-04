# Changelog

All notable changes to the Mobile App Tester Skill are documented here.

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
