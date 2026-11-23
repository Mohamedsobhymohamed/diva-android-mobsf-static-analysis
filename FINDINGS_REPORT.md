# Mobile Application Static Security Assessment: DIVA

| | |
|---|---|
| **Report date** | 2026-11-23 |
| **Assessor** | MohamedSaker (Mohamedsobhymohamed) |
| **Tool** | MobSF v4.6.9 (static analyzer) |
| **Assessment type** | Static analysis, lab environment |
| **Overall risk** | Medium (score 46/100) |

---

## 1. Executive summary

DIVA (Damn Insecure and Vulnerable App) was scanned with MobSF. The scan reported **5 high**, **7 warning** and **1 info** issues, plus 1 secure item, with a score of **46/100**. The main themes are:

- **Build and release hygiene**: debuggable build, debug signing certificate, weak signature scheme, very old minimum SDK.
- **Insecure data storage**: sensitive data in temp files and external storage, backup allowed, hardcoded secret.
- **Exposed attack surface**: exported activities and an exported content provider.
- **Injection risk**: raw SQL queries in several classes.
- **Weak native binary hardening**: no NX, stack canary or FORTIFY on the JNI library.

DIVA is intentionally vulnerable, so these results are expected. The value here is practicing how to read, triage and report them the way you would for a real app.

## 2. Scope and target

| Field | Value |
|---|---|
| App | Diva (`jakhar.aseem.diva`) |
| File | `DivaApplication.apk`, 1.43 MB |
| Version | 1.0 (code 1) |
| Target / Min SDK | 23 / 15 |
| SHA256 | `5cefc51fce9bd760b92ab2340477f4dda84b4ae0c5d04a8c9493e4fe34fab7c5` |
| SHA1 | `27e849d9d7b86a3a3357fb3e980433a91d416801` |
| MD5 | `82ab8b2193b3cfb1c737e3a786be363a` |
| Native ABIs | arm64-v8a, armeabi, armeabi-v7a, mips, mips64, x86, x86_64 |
| Trackers | 0 / 432 |

**Out of scope:** dynamic testing, network interception, runtime instrumentation, backend services.

## 3. Findings summary

| ID | Finding | Severity | Area |
|---|---|---|---|
| F-01 | Debug mode enabled | High | Build config |
| F-02 | Signed with debug certificate | High | Signing |
| F-03 | Janus vulnerability (v1-only signature) | High | Signing |
| F-04 | Installable on vulnerable Android 4.0.3-4.0.4 | High | Manifest |
| F-05 | Native libs lack NX / stack canary / FORTIFY | High | Native binary |
| F-06 | Raw SQL queries (SQL injection risk) | Warning | Code |
| F-07 | Exported activities without protection | Warning | Manifest |
| F-08 | Exported content provider without protection | Warning | Manifest |
| F-09 | Backup allowed | Warning | Manifest |
| F-10 | Sensitive data in temp file | Warning | Storage |
| F-11 | External storage read/write | Warning | Storage |
| F-12 | Hardcoded secret | Warning | Secrets |
| F-13 | Sensitive information logging | Info | Code |

Note: MobSF's dashboard summary counts (5 / 7 / 1) are grouped differently from this table, which lists each issue individually.

## 4. Detailed findings

### F-01: Debug mode enabled
- **Severity:** High
- **Evidence:** `android:debuggable=true` in the manifest; `BuildConfig.java` flagged.
- **References:** CWE-919, OWASP M1 (Improper Platform Usage), MASVS MSTG-RESILIENCE-2
- **Impact:** Lets an attacker attach a debugger, dump stack traces and access debug helpers, making reverse engineering and runtime tampering easier.
- **Fix:** Ship release builds with `debuggable=false`. Enforce in CI that release APKs are not debuggable.

### F-02: Signed with debug certificate
- **Severity:** High
- **Evidence:** Subject `C=US, O=Android, CN=Android Debug`, valid 2015-11-02 to 2045-10-25. Cert SHA256: `35d7f7ad35dfb826b70fa4b73187ed478540e32c8b8c5653b86568029fcd5840`
- **Impact:** The debug key is publicly known and not trusted for production. An attacker can sign a modified APK with an identical-looking identity.
- **Fix:** Sign production builds with a protected release key (ideally Play App Signing).

### F-03: Janus vulnerability
- **Severity:** High
- **Evidence:** v1 signature only (v2, v3, v3.1, v3.2, v4 all False).
- **Impact:** On Android 5.0-8.0, an attacker can inject code into a signed APK without breaking the signature.
- **Fix:** Sign with v2/v3 (and above) schemes, and raise the minimum SDK.

### F-04: Installable on outdated Android versions
- **Severity:** High
- **Evidence:** `minSdk=15` (Android 4.0.3-4.0.4).
- **Impact:** The app runs on devices that no longer receive security patches, so platform-level vulnerabilities are exposed.
- **Fix:** Raise `minSdkVersion`. MobSF suggests API 29+ for reasonable security update coverage.

### F-05: Weak native library hardening (`libdivajni.so`)
- **Severity:** High (NX) / Warning (others)
- **Evidence (all 7 ABIs):** NX = False, Stack canary = False (x86 shows True), FORTIFY = False, Full RELRO present, symbols stripped, no RPATH/RUNPATH.
- **Impact:** Without NX and stack canaries, memory corruption bugs in native code are easier to exploit.
- **Fix:** Compile with `-fstack-protector-strong`, `-D_FORTIFY_SOURCE=2` and a non-executable stack; use a current NDK and drop unused ABIs (mips, mips64, armeabi).

### F-06: Raw SQL queries, SQL injection risk
- **Severity:** Warning
- **Files:** `InsecureDataStorage2Activity.java`, `NotesProvider.java`, `SQLInjectionActivity.java`
- **References:** CWE-89, OWASP M7
- **Impact:** Unsanitized user input in raw queries can allow data extraction or manipulation. Sensitive data in the DB is also not encrypted.
- **Fix:** Use parameterized queries or Room; validate input; encrypt sensitive data (e.g. SQLCipher).

### F-07: Exported activities
- **Severity:** Warning
- **Components:** `jakhar.aseem.diva.APICredsActivity`, `jakhar.aseem.diva.APICreds2Activity` (explicitly exported via intent-filter)
- **Impact:** Any app on the device can launch them, potentially exposing API credential screens or flows.
- **Fix:** Set `android:exported="false"` unless required; otherwise protect with a signature-level permission and validate incoming intents.

### F-08: Exported content provider
- **Severity:** Warning
- **Component:** `jakhar.aseem.diva.NotesProvider` (`android:exported=true`)
- **Impact:** Other apps can query the provider, which combined with F-06 may expose or alter notes data.
- **Fix:** Set `exported=false` or require a permission, and restrict URI grants.

### F-09: Application backup allowed
- **Severity:** Warning
- **Evidence:** `android:allowBackup=true`
- **Impact:** With USB debugging, app data can be copied off the device via adb.
- **Fix:** Set `allowBackup=false`, or define backup rules that exclude sensitive files.

### F-10: Sensitive data in temp file
- **Severity:** Warning
- **File:** `InsecureDataStorage3Activity.java`
- **References:** CWE-276, OWASP M2, MSTG-STORAGE-2
- **Impact:** Temp files may keep sensitive data in a location with weak permissions.
- **Fix:** Do not write secrets to temp files; use app-private storage with encryption (Android Keystore).

### F-11: External storage read/write
- **Severity:** Warning
- **Files:** `InsecureDataStorage4Activity.java` (rule 00022: opens a file from an absolute path)
- **Permissions:** `READ_EXTERNAL_STORAGE`, `WRITE_EXTERNAL_STORAGE`
- **References:** CWE-276, OWASP M2, MSTG-STORAGE-2
- **Impact:** Any app with storage access can read what is written to external storage.
- **Fix:** Keep sensitive data in internal storage; use scoped storage; drop the storage permissions if not needed.

### F-12: Hardcoded secret
- **Severity:** Warning
- **Evidence:** `"pkey" : "notespin"`
- **Impact:** Anyone who decompiles the APK can read it.
- **Fix:** Do not embed secrets. Fetch from a server after authentication or store in the Android Keystore.

### F-13: Sensitive information logging
- **Severity:** Info
- **References:** CWE-532, MSTG-STORAGE-3
- **Impact:** Log output can be read via logcat on debug builds or by tools with log access.
- **Fix:** Remove or strip log statements in release builds (e.g. ProGuard/R8 rules).

## 5. Other observations

| Item | Result |
|---|---|
| Permissions | `READ_EXTERNAL_STORAGE` (dangerous), `WRITE_EXTERNAL_STORAGE` (dangerous), `INTERNET` (normal) |
| Browsable activities | None |
| Network security config | None found |
| Firebase DB exposure | None found |
| Trackers | 0 / 432 |
| Domain contacted | `payatu.com` (referenced in `APICreds2Activity.java`), status ok |
| APKiD | dx compiler, dexmerge manipulator detected |
| Android APIs used | Content Provider, IPC, native code loading, local file I/O, starting activity |

## 6. Remediation priority

1. **Immediate:** disable debug mode, sign with a release key, use v2/v3 signing (F-01, F-02, F-03).
2. **High:** fix SQL injection and lock down exported components (F-06, F-07, F-08).
3. **High:** remove insecure storage and hardcoded secrets (F-10, F-11, F-12).
4. **Medium:** raise minSdk, disable backup, harden native build (F-04, F-05, F-09).
5. **Low:** strip logging in release builds (F-13).

## 7. Limitations and next steps

- Findings come from automated static analysis and have **not been manually verified**.
- Next: dynamic analysis (Frida, Drozer, adb, Burp) to confirm exploitability of storage, SQLi, exported component and hardcoded credential issues, and to review native code behavior.

## 8. References

- MobSF: https://github.com/MobSF/Mobile-Security-Framework-MobSF
- DIVA: https://github.com/payatu/diva-android
- OWASP MASVS: https://mas.owasp.org/MASVS/
- OWASP Mobile Top 10: https://owasp.org/www-project-mobile-top-10/
- CWE: https://cwe.mitre.org/
