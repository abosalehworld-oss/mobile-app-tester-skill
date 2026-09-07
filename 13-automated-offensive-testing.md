# Phase 13: Automated Offensive Security Testing 🔴

> **Objective:** Replace manual penetration testing with automated AI-driven offensive security tools.
> The AI agent installs, configures, runs, and analyzes results from real security testing tools
> that actively attack the application — not just read its code. This is the DYNAMIC testing layer
> that complements the static analysis performed in Phases 1-12.

---

## ⚠️ IMPORTANT: This Phase is NON-NEGOTIABLE

> This phase uses **real security tools** that perform **actual attacks** against your app binary.
> It is NOT static analysis — it is dynamic offensive testing.
> **Do NOT skip this phase.** Do NOT claim "tools are unavailable" without attempting installation first.

---

## 🛠️ TOOL INSTALLATION

Before running any tests, install the required tools. Execute these commands and document the results:

### Android (APK Analysis)
```bash
# MobSF - Mobile Security Framework (Docker)
docker pull opensecurity/mobile-security-framework-mobsf
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf

# apkleaks - Find hardcoded secrets in APK
pip install apkleaks

# jadx - APK decompiler for manual verification
# Download from: https://github.com/skylot/jadx/releases
```

### iOS (IPA Analysis)
```bash
# MobSF handles IPA too via the same Docker container
# For local analysis without Docker:
pip install mobsf
```

### Flutter/React Native/Cross-platform
```bash
# Same tools apply — build the APK/IPA first, then analyze
flutter build apk --release  # Flutter
npx react-native run-android --variant=release  # React Native
```

> **If Docker is not available:** Use MobSF's hosted version or install locally via pip.
> **If pip is not available:** Document the limitation and proceed with manual checks below.

---

## 📋 PHASE 13 CHECKS

### [OT1] MobSF Automated Scan
```
Action: Upload the APK/IPA to MobSF and run full analysis
Command: curl -F 'file=@app-release.apk' http://localhost:8000/api/v1/upload
Analyze: 
  - Security Score
  - Hardcoded secrets found
  - Insecure permissions
  - Weak cryptography
  - Certificate analysis
  - Manifest analysis (Android) / Info.plist analysis (iOS)
  - Binary analysis results
  - Malware indicators
Minimum citations: 8 findings from MobSF output
```

### [OT2] Hardcoded Secrets Scan
```
Action: Scan the built binary for leaked secrets
Commands:
  apkleaks -f app-release.apk  # Android
  grep -r "API_KEY\|SECRET\|PASSWORD\|TOKEN\|private_key" --include="*.dart" --include="*.js" --include="*.kt" --include="*.swift" .
Analyze:
  - API keys in binary
  - Firebase/AWS/GCP credentials
  - Hardcoded passwords
  - OAuth client secrets
  - Encryption keys in source
Minimum citations: 3 findings
```

### [OT3] Network Security Testing
```
Action: Test the app's network communications
Commands:
  # Test SSL/TLS configuration of backend
  nmap --script ssl-enum-ciphers -p 443 [backend-host]
  # Check certificate pinning
  # Attempt MITM with self-signed cert
  openssl s_client -connect [backend-host]:443
Analyze:
  - Weak TLS versions (TLS 1.0/1.1)
  - Weak cipher suites
  - Certificate pinning effectiveness
  - Certificate expiry
Minimum citations: 3 findings
```

### [OT4] Dependency Vulnerability Scan
```
Action: Scan all dependencies for known CVEs
Commands:
  # Flutter/Dart
  dart pub outdated
  # React Native
  npm audit --json
  # Android Native
  ./gradlew dependencyCheckAnalyze
  # iOS Native
  pod audit
Analyze:
  - Critical CVEs in dependencies
  - Outdated packages with known exploits
  - Transitive dependency vulnerabilities
Minimum citations: 3 findings
```

### [OT5] Runtime Manipulation Check
```
Action: Verify app resists runtime tampering
Check:
  - Root/Jailbreak detection implemented?
  - Debugger detection implemented?
  - Code obfuscation applied? (ProGuard/R8 for Android)
  - Anti-tampering mechanisms present?
  - Integrity verification of app binary?
Minimum citations: 3 findings
```

---

## 🔧 REMEDIATION WITHIN THIS PHASE

Unlike Phases 1-12, you MAY fix Critical and High findings discovered in this phase immediately.
For each fix:
1. Show the tool output that identified the vulnerability
2. Apply the fix
3. Re-run the specific tool to verify the fix worked
4. Document before/after results

---

## ⚠️ TOOL UNAVAILABILITY PROTOCOL

If a tool genuinely cannot be installed (e.g., no Docker, no pip, restricted environment):
1. **Document exactly what you tried** and the error message
2. **Perform manual equivalent checks** using grep, find, and file analysis
3. **Mark the check as PARTIAL** in the gate report
4. **Never mark as PASS** if the tool couldn't run — mark as MANUAL_CHECK

---

## 📊 PHASE 13 GATE REPORT

```
╔══════════════════════════════════════════════════╗
║  PHASE 13 GATE — Automated Offensive Testing     ║
╠══════════════════════════════════════════════════╣
║ [OT1] MobSF Scan:           [PASS/FAIL/PARTIAL] ║
║ [OT2] Hardcoded Secrets:     [PASS/FAIL/PARTIAL] ║
║ [OT3] Network Security:     [PASS/FAIL/PARTIAL] ║
║ [OT4] Dependency Vulns:     [PASS/FAIL/PARTIAL] ║
║ [OT5] Runtime Manipulation: [PASS/FAIL/PARTIAL] ║
╠══════════════════════════════════════════════════╣
║ Tools Successfully Run:     [N/5]                ║
║ Critical Findings:          [N]                  ║
║ Findings Fixed:             [N]                  ║
║ Remaining Risks:            [N]                  ║
╠══════════════════════════════════════════════════╣
║ VERDICT: [PROCEED TO PHASE 14 / BLOCK — FIX FIRST] ║
╚══════════════════════════════════════════════════╝
```

> ⛔ **STOP — Do NOT proceed to Phase 14 until the user confirms.**

---

## 📌 MINIMUM CITATION REQUIREMENTS

| Check | Minimum Citations |
|-------|-------------------|
| OT1 - MobSF | 8 |
| OT2 - Secrets | 3 |
| OT3 - Network | 3 |
| OT4 - Dependencies | 3 |
| OT5 - Runtime | 3 |
| **Total** | **20** |

> Every citation MUST include: tool name, exact output snippet, file/component affected, and severity.
