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

## 🚨 ANTI-CHEATING RULES FOR THIS PHASE

### Rule OT-A: MANDATORY `run_command` PROOF
> Every tool execution MUST be performed via an actual `run_command` / terminal tool call.
> You MUST show the **real terminal output** — not text you wrote from memory.
> **Writing fake tool output = CHEATING.** If you cannot run the command, mark as MANUAL_CHECK.
> The user can verify by checking the tool call history in the conversation.

### Rule OT-B: MAXIMUM 2 MANUAL_CHECK ALLOWED
> You may mark **at most 2 checks** as MANUAL_CHECK due to tool unavailability.
> If 3 or more tools fail to install, you MUST troubleshoot or try alternative installation
> methods (pip, npm, brew, choco, scoop, docker, direct download).
> **Claiming all tools are unavailable = PHASE INVALID. Start over.**

### Rule OT-C: HACKER MINDSET R6 ON ALL FIXES
> If you fix any finding in this phase, you MUST apply Rule R6 (Hacker Mindset Verification):
> 1. Can the fix be bypassed?
> 2. Does the fix create a new vulnerability?
> 3. Would a penetration tester find this fix adequate?
> **No R6 verification = fix is NOT accepted.**

### Rule OT-D: MANDATORY DATABASE FRESHNESS
> Before running ANY scan, you MUST update the tool's vulnerability database first.
> Run these update commands and show the output:
> ```
> # Nuclei templates
> nuclei -update-templates
> # Trivy database
> trivy --download-db-only
> # nikto database
> nikto -update
> ```
> You MUST also run a **web search** for "latest [tool-name] version [current-year]" to verify
> you are using the most current version. Scanning with outdated databases = scanning blind.
> **If the database update fails, document the error and mark the check as PARTIAL.**

### Rule OT-E: MANDATORY FALSE POSITIVE TRIAGE (BEFORE ANY FIX)
> ⚠️ **This is the MOST CRITICAL safety rule in this phase.**
> Automated security tools produce **FALSE POSITIVES** (fake alerts). Blindly fixing a false
> positive can **BREAK the application** — disable login, corrupt data, or block real users.
>
> **BEFORE fixing ANY finding from ANY tool, you MUST complete this triage for EACH finding:**
>
> **Step 1 — Classify the finding:**
> ```
> | Finding ID | Tool | Verdict | Confidence | Reason |
> |------------|------|---------|------------|--------|
> | OT1-F1 | MobSF | TRUE_POSITIVE / FALSE_POSITIVE / UNCERTAIN | HIGH/MEDIUM/LOW | Why? |
> ```
>
> **Step 2 — MANDATORY WEB SEARCH per finding (ANTI-STALE-KNOWLEDGE):**
> Your training data is OUTDATED. You MUST NOT classify findings based on memory alone.
> For EACH finding, run a web search:
> ```
> search_web("[CVE-ID or vulnerability name] false positive [tool-name] [current-year]")
> search_web("[vulnerability name] [framework/library version] exploit proof of concept [current-year]")
> ```
> - If the internet says this CVE is **confirmed exploitable** in the detected version → TRUE_POSITIVE
> - If the internet says this is a **known false positive** for this tool/version → FALSE_POSITIVE
> - If no clear internet consensus exists → UNCERTAIN (present to user with extra caution)
> **DO NOT trust your training data over live internet results. Your knowledge may be 1-2 years stale.**
>
> **Step 3 — For each TRUE_POSITIVE or UNCERTAIN finding, explain in PLAIN LANGUAGE:**
> ```
> 🔴 FINDING: [Tool] reported [vulnerability name]
> 📍 WHERE: [file:line or endpoint]
> 👶 EXPLAIN LIKE I'M 5: [How can a hacker exploit this? What damage can they do?]
> 🔧 PROPOSED FIX: [What exactly will you change in the code?]
> ⚠️ FIX SIDE EFFECTS: [Will this fix break any feature? Will users notice anything different?]
> 🤔 FALSE POSITIVE RISK: [What is the probability this is a false alarm? Why?]
> 🌐 WEB SEARCH EVIDENCE: [What did the internet say about this CVE/vulnerability?]
> ```
>
> **Step 4 — Present the FULL triage table to the user and STOP.**
> ```
> ⛔ STOP — Do NOT fix anything until the user reviews this triage and says "fix" or "skip" for each finding.
> ```
>
> **FORBIDDEN behaviors:**
> ```
> ❌ NEVER fix a finding without completing the triage table first
> ❌ NEVER say "I fixed 5 vulnerabilities" without user approval on EACH one
> ❌ NEVER assume the user understands security jargon — ALWAYS use plain language
> ❌ NEVER hide the false positive probability — the user NEEDS this to make a decision
> ❌ NEVER batch-fix all findings at once — present them individually for approval
> ❌ NEVER classify a finding as FALSE_POSITIVE without a web search proving it
> ❌ NEVER classify a finding as TRUE_POSITIVE without a web search confirming it
> ```
>
> **WHY THIS EXISTS:** The user operating this skill may NOT be a cybersecurity expert.
> They rely on YOUR analysis to decide what to fix. If you fix a false positive, you may
> break their application. If you skip a true positive, you leave them vulnerable.
> Your training data may be outdated — a vulnerability you "remember" as harmless may now
> have a working exploit. The web search requirement ensures you use CURRENT intelligence.
> **Your triage is the user's ONLY defense. Take it seriously.**

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

> ⚠️ **You MUST complete Rule OT-E (False Positive Triage) BEFORE fixing ANY finding.**
> Do NOT skip the triage. Do NOT fix findings without explicit user approval for EACH one.

After the user approves specific findings for fixing (via Rule OT-E triage), for each approved fix:
1. Show the tool output that identified the vulnerability
2. Apply the fix
3. Re-run the specific tool to verify the fix worked
4. Apply Rule OT-C (Hacker Mindset) to verify the fix doesn't create new problems
5. Document before/after results

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
