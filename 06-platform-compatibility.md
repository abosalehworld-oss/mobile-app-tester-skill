# Phase 6: Platform Compatibility 📱

> **Objective:** Verify that the app correctly handles platform-specific behavior for
> Android and iOS. Even cross-platform frameworks (Flutter, React Native) must deal with
> platform differences in permissions, lifecycle, navigation, and native APIs.

---

## 📋 PLATFORM COMPATIBILITY CHECKS

### CHECK C1: Android-Specific Checks
```
WHAT TO CHECK:

  MANIFEST (AndroidManifest.xml):
  ❑ Are all required permissions declared?
  ❑ Are dangerous permissions requested at runtime? (not just in manifest)
  ❑ Is minSdkVersion appropriate? (≥21 for most apps, ≥23 for modern security)
  ❑ Is targetSdkVersion current? (Google Play requires recent target)
  ❑ Are intent filters correctly configured?
  ❑ Are activities/services exported only when necessary?
  ❑ Is the application label and icon set?
  ❑ Are required features declared? (<uses-feature>)

  PERMISSIONS — Runtime Handling:
  ❑ Is camera permission requested before camera use?
  ❑ Is location permission requested with rationale shown?
  ❑ Is storage permission handled for different Android versions?
    - Android 10-12: WRITE_EXTERNAL_STORAGE
    - Android 13+: READ_MEDIA_IMAGES, READ_MEDIA_VIDEO
  ❑ Is notification permission requested? (Android 13+)
  ❑ Is SCHEDULE_EXACT_ALARM permission handled? (Android 12+)
  ❑ Is the "Don't ask again" case handled? (redirect to settings)
  ❑ Is permission status checked BEFORE using the feature?

  LIFECYCLE:
  ❑ Is onSaveInstanceState/onRestoreInstanceState implemented?
  ❑ Does the app survive process death? (system kills and restores)
  ❑ Are Fragment lifecycle callbacks used correctly?
  ❑ Is configuration change (rotation) handled properly?
  ❑ Does the app handle being killed in background?

  BACK BUTTON:
  ❑ Does back button work correctly on all screens?
  ❑ Does double-back-to-exit work? (with confirmation toast)
  ❑ Are predictive back gestures supported? (Android 13+)
  ❑ Does the back button dismiss modals/dialogs correctly?

  ANDROID-SPECIFIC RED FLAGS:
  🔴 App crashes on Android 12+ due to exported components
  🔴 App requests permissions it doesn't use (Play Store rejection)
  🔴 No scoped storage support (broken on Android 10+)
  🔴 No edge-to-edge display support (Android 15+)
  🔴 No support for per-app language preferences (Android 13+)
  🔴 PendingIntent without FLAG_IMMUTABLE (Android 12+)

CITATION REQUIRED: Show AndroidManifest.xml analysis and permission handling code
```

### CHECK C2: iOS-Specific Checks
```
WHAT TO CHECK:

  INFO.PLIST:
  ❑ Are all usage descriptions provided? (NSCameraUsageDescription, etc.)
  ❑ Is the bundle ID correct and unique?
  ❑ Are URL schemes registered properly?
  ❑ Is ATS (App Transport Security) configured correctly?
  ❑ Are required device capabilities declared?
  ❑ Is the privacy manifest (PrivacyInfo.xcprivacy) present? (required 2024+)

  PERMISSIONS — iOS Style:
  ❑ Does every permission request have a clear usage description?
  ❑ Are permissions requested in context? (not all at once on launch)
  ❑ Is the Settings deep link provided when permission denied?
  ❑ Is limited photo access handled? (iOS 14+)
  ❑ Is tracking transparency handled? (ATT framework, iOS 14.5+)
  ❑ Are background modes properly declared?

  iOS NAVIGATION:
  ❑ Is swipe-back gesture supported on navigation screens?
  ❑ Are tab bars following iOS HIG (Human Interface Guidelines)?
  ❑ Are sheets/modals using iOS-style presentation?
  ❑ Is the status bar style correct (light/dark)?
  ❑ Is the home indicator area respected?
  ❑ Is Dynamic Island / notch handled properly?

  iOS-SPECIFIC RED FLAGS:
  🔴 Missing NSCameraUsageDescription → App Store rejection
  🔴 Missing privacy manifest → App Store rejection (2024+)
  🔴 App doesn't respect safe area → content under notch
  🔴 No support for iOS Dark Mode → poor user experience
  🔴 Clipboard access without user interaction → privacy warning
  🔴 Missing required background mode declarations
  🔴 Not handling iOS 15+ concentration/focus modes

CITATION REQUIRED: Show Info.plist analysis and iOS-specific handling code
```

### CHECK C3: Cross-Platform Consistency (Flutter/RN/KMP)
```
WHAT TO CHECK:
  ❑ Do platform-specific widgets adapt correctly?
    Flutter: Cupertino vs Material widgets
    RN: Platform.OS checks for different behavior
  ❑ Are platform channels/bridges implemented safely?
  ❑ Are native modules error-handled?
  ❑ Do fonts render the same on both platforms?
  ❑ Are date pickers platform-appropriate?
  ❑ Are file paths using platform separators?
  ❑ Is keyboard behavior consistent?
  ❑ Are push notifications handled for both platforms?

FLUTTER-SPECIFIC:
  ❑ Is Platform.isAndroid/isIOS used where needed?
  ❑ Are platform-specific plugins (camera, location) version-compatible?
  ❑ Is the Cupertino design used for iOS where appropriate?
  ❑ Are method channels error-handled with PlatformException?
  ❑ Are plugin versions compatible with the Flutter SDK version?

REACT NATIVE-SPECIFIC:
  ❑ Are native modules available on both platforms?
  ❑ Is Platform.select() used for platform-specific code?
  ❑ Are native bridges safe? (no crash on missing native module)
  ❑ Are Hermes-specific issues handled?
  ❑ Is the New Architecture (Fabric/TurboModules) migration considered?

COMMON CROSS-PLATFORM BUGS:
  🐛 Feature works on Android but crashes on iOS (or vice versa)
  🐛 Platform channel throws unhandled exception
  🐛 Keyboard height different on iOS vs Android, breaks layout
  🐛 Status bar overlap on one platform only
  🐛 Font rendering differently (iOS is more anti-aliased)
  🐛 Push notification payload structure differs between FCM and APNs
  🐛 Share functionality works differently on each platform
  🐛 File picker returns different path formats

CITATION REQUIRED: Show platform-specific code paths and verify both platforms
```

### CHECK C4: Device Compatibility
```
WHAT TO CHECK:
  ❑ Does the app handle different screen densities?
  ❑ Is the app compatible with foldable devices?
  ❑ Does the app work in multi-window mode? (Android)
  ❑ Does the app work with external keyboard?
  ❑ Is Bluetooth/NFC handled gracefully when not available?
  ❑ Does the app handle low storage gracefully?
  ❑ Does the app handle low memory gracefully?
  ❑ Is the app compatible with tablets? (or gracefully not)

DEVICE-SPECIFIC ISSUES:
  🐛 App crashes on devices without Google Play Services
  🐛 Camera feature crashes on devices without front camera
  🐛 NFC feature crashes when NFC is disabled
  🐛 App doesn't fit Samsung Galaxy Fold when unfolded
  🐛 App breaks in Chromebook environment
  🐛 App crashes on Android Go devices (low RAM)
  🐛 Fingerprint scanner API crashes on devices without hardware

GRACEFUL DEGRADATION CHECKS:
  ❑ Is there a feature check before using hardware features?
  ❑ Is there a fallback when a feature is not available?
  ❑ Are error messages helpful when device lacks a feature?
  ❑ Does the app run on minimum SDK version?

CITATION REQUIRED: Show hardware capability checks and fallback logic
```

### CHECK C5: Deep Linking & App Links
```
WHAT TO CHECK:
  ❑ Are deep links configured correctly?
  ❑ Do deep links navigate to the correct screen?
  ❑ Do deep links handle invalid parameters gracefully?
  ❑ Are universal links (iOS) / app links (Android) configured?
  ❑ Are deep links authenticated? (can't bypass login)
  ❑ Is deep link fallback to web configured?
  ❑ Are deep links tested with different parameter types?

COMMON BUGS:
  🐛 Deep link crashes app with invalid/missing parameters
  🐛 Deep link opens app to home screen instead of target
  🐛 Deep link opens login but doesn't redirect after auth
  🐛 Multiple apps handling same deep link scheme
  🐛 Deep link works in debug but not in release
  🐛 Universal links not configured on Apple Developer portal

CITATION REQUIRED: Show deep link configuration and handler code
```

### CHECK C6: Push Notifications
```
WHAT TO CHECK:
  ❑ Is FCM (Firebase Cloud Messaging) or APNs configured correctly?
  ❑ Is notification permission requested properly?
  ❑ Are foreground notifications handled? (showing in-app)
  ❑ Are background notifications handled? (updating data)
  ❑ Is notification tap action correct? (opens right screen)
  ❑ Are notification channels configured? (Android 8+)
  ❑ Is the notification token refreshed on change?
  ❑ Are silent/data notifications handled?

COMMON BUGS:
  🐛 Notification shows but tap does nothing
  🐛 Notification opens app to home instead of relevant screen
  🐛 Notifications not received when app is killed
  🐛 Duplicate notifications received
  🐛 Notification token not sent to server after refresh
  🐛 No notification channel → no notification on Android 8+
  🐛 Notification sounds not working
  🐛 Badge count not updating on iOS

CITATION REQUIRED: Show notification setup and handling for all states (foreground/background/killed)
```

---

## 🚦 PHASE 6 GATE — MANDATORY CHECKLIST

```
PHASE 6 GATE CHECKLIST:
  □ [C1] Android-specific checks completed
  □ [C2] iOS-specific checks completed
  □ [C3] Cross-platform consistency verified
  □ [C4] Device compatibility assessed
  □ [C5] Deep linking verified
  □ [C6] Push notifications checked
  □ Minimum 5 code citations provided
  □ Files examined list produced
  □ Platform compliance matrix produced
```

### Platform Compliance Matrix (MANDATORY):
```
┌──────────────────────────┬──────────┬──────────┐
│ Check Area               │ Android  │ iOS      │
├──────────────────────────┼──────────┼──────────┤
│ Permissions              │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
│ Lifecycle                │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
│ Navigation               │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
│ Deep Linking             │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
│ Push Notifications       │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
│ Build Configuration      │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
│ Store Compliance         │ ✅/⚠️/❌  │ ✅/⚠️/❌  │
└──────────────────────────┴──────────┴──────────┘
```

### Gate Report Format:
```
══════════════════════════════════════════════════════════
  ✅ PHASE 6 COMPLETE: Platform Compatibility
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/9 items)

  📱 Platform Health Score: [X/10]
  Android Compliance: [X/7] checks passed
  iOS Compliance: [X/7] checks passed
  Cross-Platform Issues: [count]
══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Present platform findings before proceeding to Phase 7.**
