# Phase 12: Store & Commercial Readiness 🏪

> **Objective:** Verify that the app meets ALL technical, legal, and commercial requirements
> for release on Google Play Store and Apple App Store. This is the FINAL quality gate
> before the app reaches real paying users. Covers build configuration, store policies,
> payment systems, monetization, and legal compliance.

---

## ⚠️ CRITICAL: WEB SEARCH IS MANDATORY IN THIS PHASE

> Store policies change EVERY YEAR. Google and Apple update their requirements regularly.
> Your training data may be OUTDATED. You MUST search the web for current requirements
> before completing each check.

```
🚫 DO NOT rely on your training data alone for store policies.
🚫 DO NOT guess current SDK version requirements.
🚫 DO NOT assume last year's privacy requirements are still valid.

✅ SEARCH the web for: "Google Play Store requirements [current year]"
✅ SEARCH the web for: "Apple App Store review guidelines [current year]"
✅ SEARCH the web for: "Google Play target SDK requirement [current year]"
✅ SEARCH the web for: "Apple privacy manifest requirements [current year]"
✅ CITE the source URL for every policy requirement you reference.
```

**For EVERY store policy check, you MUST include:**
- The requirement description
- The source URL where you verified it
- The date you checked it
- Whether the app meets the requirement

---

## 📋 STORE READINESS CHECKS

### CHECK R1: Build Configuration — Release Mode
```
WHAT TO CHECK:

  FLUTTER:
  ❑ Is `flutter build appbundle` used? (NOT `flutter build apk`)
  ❑ Is `--release` flag used for final build?
  ❑ Is `--obfuscate` flag configured?
  ❑ Is `--split-debug-info` configured?
  ❑ Is `kReleaseMode` used to guard debug features?
  ❑ Are `print()` / `debugPrint()` calls removed or guarded?
  ❑ Is `debugShowCheckedModeBanner: false` set?

  ANDROID (build.gradle):
  ❑ Is `minifyEnabled true` set for release?
  ❑ Is `shrinkResources true` set for release?
  ❑ Is `signingConfigs` configured with keystore?
  ❑ Is `versionCode` incremented from previous release?
  ❑ Is `versionName` set correctly?
  ❑ Is `applicationId` unique and final?
  ❑ Is `targetSdkVersion` meeting current Play Store requirement?
     → 🔍 SEARCH: "Google Play target API level requirement [current year]"
  ❑ Are ProGuard/R8 rules configured for used libraries?

  iOS (Xcode):
  ❑ Is the scheme set to Release (not Debug)?
  ❑ Is the Bundle Identifier final and matching App Store Connect?
  ❑ Is code signing configured with distribution certificate?
  ❑ Is the provisioning profile valid and not expired?
  ❑ Is `CFBundleShortVersionString` updated?
  ❑ Is `CFBundleVersion` incremented?

RED FLAGS:
  🔴 Debug signing used for release build
  🔴 `debuggable true` in release build.gradle
  🔴 Print statements logging sensitive data in release
  🔴 Test/staging API URLs in release configuration
  🔴 targetSdkVersion below Play Store minimum
  🔴 Expired provisioning profile or certificate

CITATION REQUIRED: Show build.gradle (release block) and iOS build settings
```

### CHECK R2: App Icons & Launch Screen
```
WHAT TO CHECK:

  ANDROID:
  ❑ Icon exists in all density folders?
    - mipmap-mdpi (48x48)
    - mipmap-hdpi (72x72)
    - mipmap-xhdpi (96x96)
    - mipmap-xxhdpi (144x144)
    - mipmap-xxxhdpi (192x192)
  ❑ Adaptive icon configured? (ic_launcher_foreground + background)
  ❑ Round icon variant exists?
  ❑ No default Flutter/template icon left?

  iOS:
  ❑ AppIcon.appiconset contains all required sizes?
    - 1024x1024 App Store icon (NO alpha channel)
    - All required device sizes
  ❑ No default Flutter/template icon left?

  LAUNCH SCREEN:
  ❑ Custom launch screen configured (not default white)?
  ❑ Launch screen matches app theme?

RED FLAGS:
  🔴 Default Flutter icon → unprofessional, possible rejection
  🔴 Missing icon size → store rejection
  🔴 App Store icon with alpha channel → Apple rejection

CITATION REQUIRED: List all icon files found and their sizes
```

### CHECK R3: Privacy Policy & Legal Compliance
```
⚠️ WEB SEARCH MANDATORY:
  🔍 SEARCH: "Google Play privacy policy requirements [current year]"
  🔍 SEARCH: "Apple App Store privacy requirements [current year]"
  🔍 SEARCH: "Apple required reason APIs [current year]"

WHAT TO CHECK:
  ❑ Does a privacy policy URL exist and is it accessible?
  ❑ Does the privacy policy cover ALL data the app collects?
  ❑ Is the privacy policy URL configured in the app?
  ❑ Are Terms of Service available (if needed)?
  ❑ Does the app request ONLY permissions it actually uses?
  ❑ Are unused permissions removed from manifest/plist?

  GOOGLE PLAY DATA SAFETY:
  ❑ Does the app collect personal data? → Must declare
  ❑ Does the app share data with third parties? → Must declare
  ❑ Does the app use encryption? → Must declare
  ❑ Is data deletion mechanism available?
     → 🔍 SEARCH: "Google Play data deletion requirement [current year]"

  APPLE PRIVACY:
  ❑ Is `PrivacyInfo.xcprivacy` present?
     → 🔍 SEARCH: "Apple privacy manifest requirement [current year]"
  ❑ Are all Required Reason APIs declared?
  ❑ Are App Privacy Labels accurate?
  ❑ Is ATT (App Tracking Transparency) implemented if needed?

RED FLAGS:
  🔴 No privacy policy → automatic rejection on both stores
  🔴 Privacy policy doesn't match actual data collection → legal risk
  🔴 Missing PrivacyInfo.xcprivacy → Apple rejection
  🔴 Requesting unused permissions → Play Store rejection
  🔴 No data deletion mechanism when collecting data → Play Store rejection

CITATION REQUIRED: Show privacy policy URL in code + permission list + source URLs
```

### CHECK R4: Store Listing Assets
```
WHAT TO CHECK:

  GOOGLE PLAY:
  ❑ App title (max 30 chars), short description (max 80), full description (max 4000)
  ❑ Screenshots: minimum 2 per device type
  ❑ Feature graphic (1024x500)
  ❑ App category selected
  ❑ Content rating questionnaire ready
  ❑ Contact email provided

  APPLE APP STORE:
  ❑ App name (max 30 chars), subtitle (max 30), keywords (max 100)
  ❑ Screenshots for all required device sizes
     → 🔍 SEARCH: "App Store screenshot sizes required [current year]"
  ❑ Category and age rating
  ❑ Review notes (explain app functionality to Apple reviewer)
  ❑ Demo account credentials if login required

NOTE: You cannot CREATE these assets, but you CAN check:
  - If screenshot/asset directories exist in the project
  - If store listing metadata files exist
  - If the app has enough distinct screens to screenshot
  - What is missing and needs to be created

CITATION REQUIRED: List what store assets exist and what is missing
```

---

## 💳 PAYMENT & MONETIZATION CHECKS

### CHECK R5: Payment Gateway Security & Compliance
```
⚠️ THIS IS THE HIGHEST RISK SECTION. Payment bugs = financial loss + legal liability.

⚠️ WEB SEARCH MANDATORY:
  🔍 SEARCH: "Google Play billing policy [current year]"
  🔍 SEARCH: "Apple in-app purchase guidelines [current year]"
  🔍 SEARCH: "[payment SDK name] security best practices"

WHAT TO CHECK FOR ALL PAYMENT TYPES:

  A. IN-APP PURCHASES (Digital Goods — Google Play Billing / StoreKit):
  ❑ Is official store billing API used? (NOT third-party for digital goods)
  ❑ Are purchase receipts validated SERVER-SIDE? (not just client)
  ❑ Is "Restore Purchases" available? (Apple REQUIRES this)
  ❑ Are subscription terms clearly displayed before purchase?
  ❑ Is subscription management link accessible?
  ❑ Is there a free trial? → Is trial period clearly stated?
  ❑ Are grace periods handled for failed payments?
  ❑ Is sandbox/test mode disabled in production?

  B. EXTERNAL PAYMENTS (Physical Goods/Services — Stripe/PayPal/etc.):
  ❑ Is a TRUSTED payment SDK used? (Stripe, PayPal, Square, Razorpay, etc.)
  ❑ Is the SDK version up to date?
     → 🔍 SEARCH: "[SDK name] latest version security advisory"
  ❑ Is PCI compliance handled by the SDK? (you should NEVER handle raw card data)
  ❑ Are API keys stored securely? (NOT hardcoded in source)
  ❑ Is the payment flow over HTTPS exclusively?
  ❑ Are Webhook signatures verified?
  ❑ Is idempotency implemented? (prevent double charges)
  ❑ Are payment confirmations sent to user?
  ❑ Is refund flow implemented?

  C. PAYMENT SECURITY AUDIT:
  ❑ Is card data NEVER stored locally? (SDK handles tokenization)
  ❑ Is card data NEVER logged? (not in console, analytics, or crash reports)
  ❑ Is the payment UI on a secure screen? (screenshot prevention)
  ❑ Is amount validation done server-side? (client can be tampered)
  ❑ Are failed payment retries limited? (prevent brute force)
  ❑ Is currency handling correct? (use integers/cents, not floats)
  ❑ Are international currencies supported correctly?
  ❑ Is tax calculation correct for target markets?

  D. PAYMENT FLOW VERIFICATION:
  Trace the ENTIRE payment flow and verify each step:
    1. User selects item/plan → price shown correctly?
    2. User initiates payment → loading state shown?
    3. Payment SDK processes → timeout handled?
    4. Success → receipt stored? confirmation shown? backend notified?
    5. Failure → clear error message? retry option? no charge?
    6. Network loss during payment → state preserved? no double charge?
    7. App killed during payment → recovery on restart?

COMMON PAYMENT BUGS (CRITICAL):
  🔴 Double charge on retry (no idempotency key)
  🔴 Amount calculated on client-side (can be tampered by attacker)
  🔴 Receipt validation on client only (can be bypassed)
  🔴 Raw card number logged in crash report (PCI violation)
  🔴 Test/sandbox payment keys in production
  🔴 Missing webhook signature verification (anyone can fake payment success)
  🔴 Float arithmetic for currency (0.1 + 0.2 ≠ 0.3 → wrong prices)
  🔴 No offline handling → payment state lost
  🔴 Subscription renewal not handled → user loses access
  🔴 Using external payment for digital goods → STORE REJECTION

SEARCH PATTERNS:
  - "sk_test_", "pk_test_" (Stripe test keys)
  - "sandbox" (PayPal sandbox mode)
  - "test_mode", "debug_mode" in payment code
  - Credit card regex patterns (card numbers being captured/logged)
  - "price", "amount", "cost" calculated on client

CITATION REQUIRED: Trace complete payment flow with security assessment at each step
```

### CHECK R6: Revenue & Monetization System
```
WHAT TO CHECK (if app has ads, subscriptions, or revenue features):

  A. ADVERTISING:
  ❑ Is ad SDK properly integrated? (AdMob, Meta Audience Network, etc.)
  ❑ Is ad SDK initialized AFTER user consent? (GDPR/CCPA)
  ❑ Are test ads disabled in production?
  ❑ Is ad frequency reasonable? (not every 5 seconds → bad UX + rejection)
  ❑ Are ads not covering critical UI elements?
  ❑ Is ad content filtering enabled? (block inappropriate ads)
  ❑ Are ads paused when app is in background? (battery drain)
  ❑ Are rewarded ads delivering rewards correctly?
  ❑ Is ad revenue tracking accurate?

  B. SUBSCRIPTION MANAGEMENT:
  ❑ Is subscription status checked on app launch? (not just cached)
  ❑ Is subscription expiry handled? (downgrade to free gracefully)
  ❑ Is subscription upgrade/downgrade supported?
  ❑ Are promotional offers handled correctly?
  ❑ Is subscription sharing (Family Sharing) handled?
  ❑ Is subscription status synced across devices?

  C. REVENUE PROTECTION:
  ❑ Is premium content gated SERVER-SIDE? (not just UI hidden)
  ❑ Can free users access paid features by modifying client code?
  ❑ Is receipt/purchase token validated on backend before granting access?
  ❑ Are subscription status checks tamper-proof?
  ❑ Is there a grace period for payment failures?

RED FLAGS:
  🔴 Premium check only on client → easily bypassed
  🔴 Ad SDK initialized before consent → GDPR violation + store rejection
  🔴 Test ad unit IDs in production → no revenue
  🔴 Subscription status only cached locally → can be faked
  🔴 No server-side purchase validation → revenue theft

CITATION REQUIRED: Show monetization integration and access control code
```

---

## 🌍 ENVIRONMENT & MARKET CHECKS

### CHECK R7: Environment & API Configuration
```
WHAT TO CHECK:
  ❑ Is the app pointing to PRODUCTION API? (not staging/dev)
  ❑ Are test/mock data removed?
  ❑ Are debug endpoints disabled?
  ❑ Are API keys for production (not test keys)?
  ❑ Are feature flags set to production values?
  ❑ Are analytics pointing to production project?
  ❑ Is crash reporting pointing to production project?

SEARCH PATTERNS:
  - "staging", "dev", "test", "localhost", "127.0.0.1", "10.0.2.2"
  - "sk_test_", "pk_test_" (Stripe test keys)
  - "sandbox" (PayPal sandbox)
  - "debug", "DEBUG", "isDebug"
  - ".env.development", ".env.staging"
  - "TODO", "FIXME", "HACK", "XXX"

RED FLAGS:
  🔴 API base URL contains "staging" or "dev"
  🔴 Payment test keys in production build
  🔴 Firebase project is test/dev project
  🔴 Mock data visible to users
  🔴 Debug menu accessible in release
  🔴 TODO comments indicating unfinished features

CITATION REQUIRED: Show ALL base URLs and API key configurations
```

### CHECK R8: Localization & Market Readiness
```
WHAT TO CHECK:
  ❑ Is the app language consistent? (not mixing Arabic and English randomly)
  ❑ Are all user-facing strings localizable?
  ❑ Is RTL (Right-to-Left) supported if targeting Arabic/Hebrew markets?
  ❑ Are date/time/currency formats locale-aware?
  ❑ Is the app name appropriate in target markets?

  CURRENCY HANDLING (CRITICAL for payment apps):
  ❑ Are prices displayed in local currency?
  ❑ Is currency symbol correct for locale? ($ vs € vs ر.س)
  ❑ Are decimal separators correct? (1,000.00 vs 1.000,00)
  ❑ Is currency conversion handled server-side? (not client)

RED FLAGS:
  🔴 Mixing languages randomly in UI
  🔴 Hardcoded "$" for non-US market
  🔴 Date as MM/DD in European market
  🔴 RTL text broken in LTR layout

CITATION REQUIRED: Show language config and any hardcoded strings
```

---

## 🚦 PHASE 12 GATE — MANDATORY CHECKLIST

```
PHASE 12 GATE CHECKLIST:
  □ [R1] Release build configuration verified (Android + iOS)
  □ [R2] App icons and launch screen verified
  □ [R3] Privacy policy and legal compliance checked
  □ [R4] Store listing assets inventory produced
  □ [R5] Payment gateway security fully audited (if applicable)
  □ [R6] Revenue/monetization system verified (if applicable)
  □ [R7] Production environment configuration verified
  □ [R8] Localization and market readiness checked
  □ Web search performed for current store policies (URLs cited)
  □ Minimum 8 code citations provided
  □ Store readiness matrix produced
  □ Payment flow traced end-to-end (if applicable)
```

### Store Readiness Matrix (MANDATORY):

**Template (copy and fill):**

```markdown
## Store Readiness Matrix

| Check Area | Google Play | App Store | Status |
|-----------|------------|-----------|--------|
| Build Config (Release) | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| App Icons (All Sizes) | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| Privacy Policy | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| Privacy Manifest | N/A | ✅/⚠️/❌ | [detail] |
| Data Safety / Labels | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| Store Listing Assets | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| Payment Compliance | ✅/⚠️/❌/N/A | ✅/⚠️/❌/N/A | [detail] |
| Payment Security | ✅/⚠️/❌/N/A | ✅/⚠️/❌/N/A | [detail] |
| Revenue Protection | ✅/⚠️/❌/N/A | ✅/⚠️/❌/N/A | [detail] |
| Production Environment | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| Localization | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
| Code Signing | ✅/⚠️/❌ | ✅/⚠️/❌ | [detail] |
```

### Gate Report Format:

```markdown
## ✅ Phase 12 Complete: Store & Commercial Readiness

| Metric | Value |
|--------|-------|
| Findings | [X] Critical, [Y] High, [Z] Medium, [W] Low |
| Gate Status | [PASSED/FAILED] ([checked]/12 items) |
| Google Play Ready | [YES / NO / PARTIAL] |
| App Store Ready | [YES / NO / PARTIAL] |
| Payment System Secure | [YES / NO / N/A] |
| Revenue System Secure | [YES / NO / N/A] |
| Privacy Compliant | [YES / NO / PARTIAL] |
| Web Sources Cited | [count] URLs |

> **COMMERCIAL RELEASE VERDICT: [🟢 READY / 🟡 FIX REQUIRED / 🔴 NOT READY]**
```

---

### ⛔ FINAL STOP POINT
**This is the LAST phase before commercial release. The AI must give a clear
YES or NO verdict. If NO, list EXACTLY what must be fixed. If YES, the user
can proceed to store submission with confidence.**

**After this phase, if fixes are needed, use Phase 11 (Structured Remediation)
to fix all findings, then RE-RUN Phase 12 to verify fixes.**
