# Phase 4: Security Audit 🔒

> **Objective:** Perform a comprehensive security audit based on the OWASP Mobile Top 10,
> platform-specific security guidelines, and real-world mobile app vulnerability databases.
> This is the MOST CRITICAL phase — security bugs can lead to data breaches, financial loss,
> and legal liability.

---

## ⚠️ SECURITY IS NON-NEGOTIABLE

> Every security finding in this phase is automatically classified as 🔴 CRITICAL or 🟠 HIGH.
> There are NO low-severity security issues. If it's a security concern, it matters.

---

## 📋 SECURITY CHECKS — OWASP MOBILE TOP 10 (2024)

### CHECK S1: M1 — Improper Credential Usage
```
WHAT TO CHECK:
  ❑ Are API keys hardcoded in source code?
  ❑ Are passwords/secrets stored in plain text?
  ❑ Are credentials logged to console or crash reports?
  ❑ Are service account keys embedded in the app?
  ❑ Are OAuth client secrets in the client code? (should be server-side)
  ❑ Are Firebase/AWS/Azure keys exposed?

SEARCH PATTERNS (run these across entire codebase):
  - "api_key", "apiKey", "API_KEY"
  - "secret", "SECRET", "client_secret"
  - "password", "PASSWORD", "passwd"
  - "token" (check if hardcoded vs dynamically fetched)
  - "Bearer ", "Basic " (hardcoded auth headers)
  - "sk_live_", "pk_live_" (Stripe keys)
  - "AIza" (Google API keys)
  - "AKIA" (AWS access keys)
  - "-----BEGIN PRIVATE KEY-----"
  - "-----BEGIN RSA PRIVATE KEY-----"
  - Firebase config objects with direct project IDs
  - .env files committed to repository

SEVERITY: 🔴 CRITICAL — Any hardcoded secret is an immediate breach risk
CITATION REQUIRED: Show EVERY instance of hardcoded secrets or prove none exist
```

### CHECK S2: M2 — Inadequate Supply Chain Security
```
WHAT TO CHECK:
  ❑ Are all dependencies from trusted sources?
  ❑ Are dependency versions pinned? (not "latest" or "^" with wide range)
  ❑ Are there dependencies with known CVEs?
  ❑ Is there a lock file committed? (pubspec.lock, package-lock.json, Podfile.lock)
  ❑ Are there typo-squatting risks? (similar name to popular package)
  ❑ Are there dependencies pulling from non-standard registries?
  ❑ Are native library (.so, .a, .dylib) sources verified?

HOW TO CHECK:
  1. List all dependencies with versions
  2. For each, check:
     - Is the package well-known? (pub.dev/npm downloads)
     - When was it last updated?
     - Is the source repository accessible?
  3. Look for any dependency that:
     - Has fewer than 100 downloads/likes
     - Was last updated more than 2 years ago
     - Has no documentation
     - Is a fork of a popular package (potential supply chain attack)

SEVERITY: 🟠 HIGH — Compromised dependencies can inject malicious code
CITATION REQUIRED: List all dependencies with risk assessment
```

### CHECK S3: M3 — Insecure Authentication/Authorization
```
WHAT TO CHECK:

  AUTHENTICATION:
  ❑ Is authentication over HTTPS only?
  ❑ Is multi-factor authentication (MFA) supported?
  ❑ Are password requirements enforced? (length, complexity)
  ❑ Is brute-force protection implemented? (rate limiting, lockout)
  ❑ Is biometric auth implemented correctly? (not just a local flag)
  ❑ Are session tokens cryptographically random?
  ❑ Is "remember me" token separate from session token?
  ❑ Is there proper account lockout after failed attempts?

  AUTHORIZATION:
  ❑ Are role checks performed server-side? (not just hidden UI elements)
  ❑ Can a regular user access admin API endpoints?
  ❑ Is there Insecure Direct Object Reference (IDOR) risk?
    (User A accessing User B's data by changing ID in request)
  ❑ Are deep links to protected content guarded?
  ❑ Is there horizontal privilege escalation risk?

COMMON VULNERABILITIES:
  🔴 Password sent in URL query parameters (logged by servers)
  🔴 Token stored in SharedPreferences/UserDefaults (unencrypted)
  🔴 Biometric bypass: app stores "isAuthenticated = true" locally
  🔴 JWT token never expires (no expiration claim)
  🔴 Refresh token sent to wrong endpoint
  🔴 User ID taken from client request instead of server session
  🔴 Role check done only in UI (if isAdmin → show button)
  🔴 OAuth redirect_uri not validated → token theft

CITATION REQUIRED: Trace the complete authentication flow with security assessment
```

### CHECK S4: M4 — Insufficient Input/Output Validation
```
WHAT TO CHECK:
  ❑ Is user input sanitized before use?
  ❑ Is SQL injection possible? (raw queries with user input)
  ❑ Is XSS possible? (user input rendered as HTML in WebView)
  ❑ Is path traversal possible? (user input in file paths)
  ❑ Is command injection possible? (user input in system commands)
  ❑ Are deep link parameters validated?
  ❑ Is file upload validated? (type, size, content)
  ❑ Is clipboard content validated before use?

COMMON VULNERABILITIES:
  🔴 db.rawQuery("SELECT * FROM users WHERE name = '$userInput'")
     → SQL injection: userInput = "'; DROP TABLE users; --"

  🔴 WebView.loadHtml("<h1>$userName</h1>")
     → XSS: userName = "<script>stealCookies()</script>"

  🔴 File.open("/data/files/$userInput")
     → Path traversal: userInput = "../../etc/passwd"

  🔴 Deep link: myapp://profile?user_id=$UNTRUSTED
     → IDOR or crash with invalid ID

  🔴 JSON.parse(untrustedInput) without try-catch
     → Crash on malformed input

  🔴 Intent/URL scheme handling without origin validation
     → Malicious app can trigger actions

INPUT VALIDATION CHECKLIST:
  For EVERY user input in the app:
  □ Is there max length?
  □ Is there format validation (regex)?
  □ Is there type checking?
  □ Are special characters escaped/rejected?
  □ Is the input used in a dangerous context (SQL, HTML, file path)?

CITATION REQUIRED: Show every point where user input enters the system
```

### CHECK S5: M5 — Insecure Communication
```
WHAT TO CHECK:
  ❑ Are ALL network calls over HTTPS? (no HTTP fallback)
  ❑ Is certificate pinning implemented?
  ❑ Is certificate validation NOT disabled? (common during development)
  ❑ Are WebSocket connections over WSS? (not WS)
  ❑ Is the app vulnerable to man-in-the-middle (MITM) attacks?
  ❑ Are network security configurations proper?
  ❑ Is data encrypted in transit?

SEARCH PATTERNS:
  - "http://" (should be "https://")
  - "allowSelfSigned", "trustAllCertificates"
  - "CERTIFICATE_VERIFY_NONE", "TrustStrategy", "X509TrustManager"
  - "cleartext", "cleartextTrafficPermitted"
  - "NSAllowsArbitraryLoads" (iOS Info.plist)
  - "android:usesCleartextTraffic=\"true\"" (AndroidManifest.xml)
  - "badCertificateCallback" returning true
  - WebSocket "ws://" instead of "wss://"

ANDROID-SPECIFIC:
  ❑ network_security_config.xml exists?
  ❑ cleartextTrafficPermitted set to false?
  ❑ Custom trust anchors properly configured?

iOS-SPECIFIC:
  ❑ App Transport Security (ATS) enabled?
  ❑ NSAllowsArbitraryLoads set to false?
  ❑ Exception domains minimized?

SEVERITY: 🔴 CRITICAL — HTTP traffic can be intercepted on any public WiFi
CITATION REQUIRED: Show all network configuration files and base URL definitions
```

### CHECK S6: M6 — Inadequate Privacy Controls
```
WHAT TO CHECK:
  ❑ What personal data does the app collect?
  ❑ Is data collection minimized? (only what's needed)
  ❑ Is there a privacy policy accessible from the app?
  ❑ Is user consent obtained before data collection?
  ❑ Can users request data deletion?
  ❑ Is PII (Personally Identifiable Information) logged?
  ❑ Are analytics collecting more than needed?
  ❑ Is location tracking transparent?
  ❑ Is clipboard access justified?
  ❑ Are third-party SDKs respecting user privacy?

SEARCH PATTERNS:
  - Log/print statements containing user data (email, name, phone)
  - Analytics events with PII
  - Crash reports with user data
  - Screenshot/screen recording not disabled on sensitive screens
  - Clipboard access without user action

PII DATA TYPES TO CHECK:
  - Email addresses
  - Phone numbers
  - Physical addresses
  - Device identifiers (IMEI, MAC address)
  - Location data
  - Financial information
  - Health data
  - Biometric data
  - Contacts access
  - Camera/microphone access

CITATION REQUIRED: List all PII collected and how it's handled
```

### CHECK S7: M7 — Insufficient Binary Protections
```
WHAT TO CHECK:
  ❑ Is code obfuscation enabled? (ProGuard/R8 for Android, bitcode for iOS)
  ❑ Is root/jailbreak detection implemented?
  ❑ Is tampering detection implemented? (app signature verification)
  ❑ Is debugging detection in place? (anti-debug)
  ❑ Are release builds stripped of debug information?
  ❑ Is reverse engineering protection in place?

ANDROID-SPECIFIC:
  ❑ ProGuard/R8 rules configured in build.gradle?
  ❑ minifyEnabled set to true for release?
  ❑ shrinkResources enabled?
  ❑ debuggable set to false for release?

iOS-SPECIFIC:
  ❑ Bitcode enabled?
  ❑ Strip Swift Symbols enabled?
  ❑ Debug Information Format set correctly for release?

FLUTTER-SPECIFIC:
  ❑ --obfuscate flag used in release build?
  ❑ --split-debug-info used?
  ❑ kReleaseMode checks for debug features?
  ❑ assert() used for debug-only code?

CITATION REQUIRED: Show build configuration files (build.gradle, Podfile, etc.)
```

### CHECK S8: M8 — Security Misconfiguration
```
WHAT TO CHECK:
  ❑ Are default credentials changed? (admin/admin)
  ❑ Are debug features disabled in production?
  ❑ Are stack traces hidden from users?
  ❑ Is the backup flag configured correctly?
  ❑ Are exported components secured? (Android)
  ❑ Is the minimum SDK version secure enough?
  ❑ Are unused permissions removed?

ANDROID-SPECIFIC:
  ❑ android:allowBackup="false" (or encrypted backup)?
  ❑ android:exported="false" for internal components?
  ❑ Content providers properly secured?
  ❑ Intent filters not overly broad?
  ❑ No unnecessary permissions declared?

iOS-SPECIFIC:
  ❑ Info.plist configurations reviewed?
  ❑ Keychain access groups properly configured?
  ❑ URL schemes not hijackable?

COMMON MISCONFIGURATIONS:
  🔴 android:debuggable="true" in release
  🔴 android:allowBackup="true" without encryption
  🔴 All activities exported (accessible by other apps)
  🔴 Content provider without permissions (data accessible to all apps)
  🔴 Minimum SDK too low (missing security features)
  🔴 WebView with JavaScript enabled on untrusted content
  🔴 React Native debug bridge left enabled in production

CITATION REQUIRED: Show AndroidManifest.xml / Info.plist analysis
```

### CHECK S9: M9 — Insecure Data Storage
```
WHAT TO CHECK:
  ❑ Where is sensitive data stored?
  ❑ Is encryption used for local data at rest?
  ❑ Are proper storage mechanisms used?

STORAGE SECURITY MATRIX:
  ┌─────────────────────────┬──────────────────┬───────────────────┐
  │ Data Type               │ ❌ INSECURE        │ ✅ SECURE           │
  ├─────────────────────────┼──────────────────┼───────────────────┤
  │ Auth tokens             │ SharedPrefs      │ Keystore/Keychain │
  │ Passwords               │ Plain text file  │ Never store local │
  │ Credit card numbers     │ SQLite           │ Never store local │
  │ User PII                │ Unencrypted DB   │ Encrypted DB      │
  │ Session data            │ Cookies in WebView│ Secure storage    │
  │ Encryption keys         │ Hardcoded        │ Hardware-backed   │
  │ App settings (non-sens) │ —                │ SharedPrefs OK    │
  └─────────────────────────┴──────────────────┴───────────────────┘

SEARCH PATTERNS:
  Flutter:
    - "SharedPreferences" with sensitive keys (token, password, secret)
    - "File" operations writing sensitive data
    - "sqflite" / "hive" without encryption

  React Native:
    - "AsyncStorage" with sensitive keys
    - "MMKV" without encryption
    - "@react-native-async-storage"

  Android Native:
    - "getSharedPreferences" with MODE_WORLD_READABLE
    - "openFileOutput" with MODE_WORLD_READABLE
    - SQLite without SQLCipher

  iOS Native:
    - "UserDefaults" for sensitive data
    - "FileManager" without encryption
    - Keychain without proper access control

ADDITIONAL CHECKS:
  ❑ Is the app's data directory encrypted? (file-level encryption)
  ❑ Are database files encrypted? (SQLCipher, encrypted Hive)
  ❑ Are log files containing sensitive data?
  ❑ Are cached network responses containing sensitive data?
  ❑ Is screenshot prevention implemented for sensitive screens?
  ❑ Are temporary files cleaned up?

CITATION REQUIRED: Map ALL local storage usage with sensitivity assessment
```

### CHECK S10: M10 — Insufficient Cryptography
```
WHAT TO CHECK:
  ❑ Are cryptographic algorithms up to date?
  ❑ Are key sizes sufficient?
  ❑ Are random numbers cryptographically secure?
  ❑ Is encryption implemented correctly?

APPROVED ALGORITHMS:
  ✅ AES-256-GCM (symmetric encryption)
  ✅ RSA-2048+ (asymmetric encryption)
  ✅ SHA-256+ (hashing)
  ✅ PBKDF2/bcrypt/Argon2 (password hashing)
  ✅ ECDSA/Ed25519 (digital signatures)
  ✅ TLS 1.2+ (transport encryption)

DEPRECATED/INSECURE:
  ❌ MD5 (broken hash)
  ❌ SHA-1 (collision attacks)
  ❌ DES/3DES (weak encryption)
  ❌ RC4 (broken cipher)
  ❌ ECB mode (pattern leakage)
  ❌ AES-CBC without HMAC (padding oracle attack)
  ❌ Random() / Math.random() for security (not cryptographic)
  ❌ Base64 (encoding, NOT encryption!)

COMMON MISTAKES:
  🔴 Using Base64 as "encryption" (it's just encoding)
  🔴 Hardcoded encryption key in source code
  🔴 Using MD5 for password hashing
  🔴 Using Random() instead of SecureRandom()
  🔴 Rolling own cryptography instead of using established libraries
  🔴 ECB mode encryption (identical blocks produce identical cipher text)
  🔴 IV (initialization vector) reuse
  🔴 Not using authenticated encryption (AES-GCM)

CITATION REQUIRED: Show all cryptographic code and verify algorithm safety
```

---

## 🚦 PHASE 4 GATE — MANDATORY CHECKLIST

```
PHASE 4 GATE CHECKLIST:
  □ [S1]  Hardcoded credentials search completed
  □ [S2]  Supply chain / dependency security assessed
  □ [S3]  Authentication/authorization flow audited
  □ [S4]  Input validation checked on all entry points
  □ [S5]  Network communication security verified
  □ [S6]  Privacy controls and PII handling reviewed
  □ [S7]  Binary protection configuration checked
  □ [S8]  Security misconfiguration scan completed
  □ [S9]  Local data storage security verified
  □ [S10] Cryptographic implementations audited
  □ Minimum 10 code citations provided
  □ Files examined list produced
  □ Cross-reference to Phase 1 dependency issues
  □ OWASP compliance matrix produced
```

### OWASP Compliance Matrix (MANDATORY):
```
┌────────────────────────────────────┬─────────┬──────────┐
│ OWASP M# - Category               │ Status  │ Findings │
├────────────────────────────────────┼─────────┼──────────┤
│ M1 - Improper Credentials         │ ✅/⚠️/❌ │ [count]  │
│ M2 - Supply Chain                 │ ✅/⚠️/❌ │ [count]  │
│ M3 - Insecure Auth                │ ✅/⚠️/❌ │ [count]  │
│ M4 - Input Validation             │ ✅/⚠️/❌ │ [count]  │
│ M5 - Insecure Communication       │ ✅/⚠️/❌ │ [count]  │
│ M6 - Privacy Controls             │ ✅/⚠️/❌ │ [count]  │
│ M7 - Binary Protections           │ ✅/⚠️/❌ │ [count]  │
│ M8 - Security Misconfiguration    │ ✅/⚠️/❌ │ [count]  │
│ M9 - Insecure Data Storage        │ ✅/⚠️/❌ │ [count]  │
│ M10 - Insufficient Crypto         │ ✅/⚠️/❌ │ [count]  │
└────────────────────────────────────┴─────────┴──────────┘
```

### Gate Report Format:
```
══════════════════════════════════════════════════════════
  ✅ PHASE 4 COMPLETE: Security Audit
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/14 items)

  🔒 Security Health Score: [X/10]
  OWASP Compliance: [X/10] categories passed
  Hardcoded Secrets: [count found]
  Vulnerable Dependencies: [count found]
  Unvalidated Inputs: [count found]
  Insecure Storage: [count found]
══════════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**Security findings are the HIGHEST PRIORITY. Present ALL findings and get user
acknowledgment before proceeding. Any CRITICAL finding should be flagged for
IMMEDIATE remediation.**
