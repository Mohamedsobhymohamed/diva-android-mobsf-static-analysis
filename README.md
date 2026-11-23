# DIVA Android Static Analysis with MobSF
 
> Static security assessment of **DIVA (Damn Insecure and Vulnerable App)** using Mobile Security Framework (MobSF) v4.6.9, with findings mapped to CWE, OWASP Mobile Top 10 and OWASP MASVS.
 
![Platform](https://img.shields.io/badge/platform-Android-green)
![Tool](https://img.shields.io/badge/tool-MobSF%20v4.6.9-blue)
![Analysis](https://img.shields.io/badge/analysis-static-orange)
![Score](https://img.shields.io/badge/MobSF%20score-46%2F100-yellow)
![Risk](https://img.shields.io/badge/risk-medium-yellow)
 
---
 
## Overview
 
DIVA is an intentionally vulnerable Android app built for security training. This repo documents a static analysis of the APK: what was scanned, what MobSF found, why each finding matters, and how to fix it.
 
This is a **learning / portfolio project** done on a deliberately vulnerable training app, in a lab environment. Nothing here targets a real product.
 
## Target
 
| Field | Value |
|---|---|
| App name | Diva |
| Package | `jakhar.aseem.diva` |
| Main activity | `jakhar.aseem.diva.MainActivity` |
| File | `DivaApplication.apk` (1.43 MB) |
| Version | 1.0 (code 1) |
| SDK | Target 23, Min 15 |
| SHA256 | `5cefc51fce9bd760b92ab2340477f4dda84b4ae0c5d04a8c9493e4fe34fab7c5` |
| MD5 | `82ab8b2193b3cfb1c737e3a786be363a` |
| Languages | Java, C/C++ (native lib `libdivajni.so`) |
 
## Results at a glance
 
| Metric | Value |
|---|---|
| Security score | **46 / 100** (Medium Risk) |
| High | 5 |
| Warning | 7 |
| Info | 1 |
| Secure | 1 |
| Trackers | 0 / 432 |
| Exported components | 2 of 17 activities, 1 of 1 provider |
 
### Key findings
 
| # | Finding | Severity | Reference |
|---|---|---|---|
| 1 | Debug enabled (`android:debuggable=true`) | High | CWE-919, M1, MSTG-RESILIENCE-2 |
| 2 | Signed with debug certificate | High | Certificate analysis |
| 3 | Vulnerable to Janus (v1 signature only) | High | Certificate analysis |
| 4 | Installable on Android 4.0.3-4.0.4 (minSdk 15) | High | Manifest analysis |
| 5 | Raw SQL queries (SQL injection risk) | Warning | CWE-89, M7 |
| 6 | Exported activities `APICredsActivity`, `APICreds2Activity` | Warning | Manifest analysis |
| 7 | Exported content provider `NotesProvider` | Warning | Manifest analysis |
| 8 | `allowBackup=true` | Warning | Manifest analysis |
| 9 | Sensitive data written to temp file | Warning | CWE-276, M2, MSTG-STORAGE-2 |
| 10 | External storage read/write | Warning | CWE-276, M2, MSTG-STORAGE-2 |
| 11 | Hardcoded secret (`pkey`) | Warning | Secrets detection |
| 12 | Native libs without NX, stack canary or FORTIFY | High / Warning | Binary analysis |
| 13 | Application logs information | Info | CWE-532, MSTG-STORAGE-3 |
 
Full details, impact and remediation are in **[FINDINGS_REPORT.md](FINDINGS_REPORT.md)**.
 
## Repo structure
 
```
.
├── README.md             # This file
├── FINDINGS_REPORT.md    # Detailed findings, risk rating, remediation
└── DivaApplication.apk   # Diva file
```
 
## Methodology
 
1. Obtained `DivaApplication.apk` (public training app)
2. Uploaded it to MobSF v4.6.9 (static analyzer)
3. Reviewed: signer certificate, permissions, manifest, code analysis, native binary analysis, secrets, URLs
4. Mapped each finding to CWE / OWASP Mobile Top 10 / MASVS
5. Wrote remediation guidance
## Reproduce it
 
```bash
# 1. Run MobSF with Docker
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
 
# 2. Open http://localhost:8000 and upload DivaApplication.apk
 
# 3. Export the PDF report from the Static Analyzer page
```
 
DIVA source and APK: https://github.com/payatu/diva-android
 
## Scope and limitations
 
- **Static analysis only.** No dynamic testing or exploitation was done in this report. Findings are scanner output and need manual confirmation.
- MobSF can produce false positives and misses logic flaws.
- Next step: dynamic analysis with Frida / Drozer / adb to confirm and exploit each issue (insecure storage, SQLi, exported components, hardcoded credentials, input validation, native code).
## Skills demonstrated
 
Mobile application security, static analysis, MobSF, Android manifest and component review, secure storage review, OWASP MASVS / Mobile Top 10 mapping, CWE classification, risk rating and security reporting.
 
## Disclaimer
 
For educational purposes only. DIVA is an intentionally vulnerable app, and all testing was done on my own lab setup. Only test applications you own or have written permission to test.
 
## Author
 
**MohamedSaker** | [@Mohamedsobhymohamed](https://github.com/Mohamedsobhymohamed)
