# Mobile Security Analysis — DIVA Android
**Static & External Threat Analysis with Yaazhini and BeVigil**

> Academic Lab | S2 — Mobile Programming Security  
> Analyst: Laila Elamiri | Date: 15-MAY-2026

---

## Overview

This project performs a full mobile application security assessment of the intentionally vulnerable Android application **DIVA Android** (`jakhar.aseem.diva`), using two complementary analysis approaches:

- **Yaazhini** — Internal static analysis (APK reverse engineering, manifest inspection, source code review)
- **BeVigil** — External threat intelligence (exposed endpoints, hardcoded IPs, API surface mapping)

The analysis identified **12 security findings** across 4 OWASP MASVS categories, with an overall risk level rated **HIGH**.

---

## Target Application

| Field | Value |
|-------|-------|
| Package | `jakhar.aseem.diva` |
| Name | DIVA Android (Damn Insecure and Vulnerable App) |
| Version | 1.0 |
| Min SDK | 15 |
| Target SDK | 23 |
| Size | 1.4 MB |

---

## Repository Structure

```
lab-mobile-security/
│
├── 00-scope/           # Target definition and analysis objectives
├── 01-bevigil/         # BeVigil external analysis outputs and notes
├── 02-yaazhini/        # Yaazhini static analysis outputs and screenshots
├── 03-triage/          # Finding triage and false positive analysis
├── 04-report/          # Final report and OWASP mapping
│
├── analyse_info.txt    # Raw analysis metadata
├── commands.txt        # Commands used during the lab
└── rapport_final.md    # Full security report (French)
```

---

## Methodology

```
1. SCOPE DEFINITION
   Target APK identified, analysis boundaries defined.

2. EXTERNAL ANALYSIS — BeVigil
   APK uploaded to BeVigil platform.
   Extracted: endpoints, hardcoded IPs, secrets, cloud URLs.

3. STATIC ANALYSIS — Yaazhini
   APK decompiled locally.
   Inspected: manifest, Java source code, permissions, storage.

4. TRIAGE
   Findings classified, true positives confirmed, false positives eliminated.

5. REPORTING
   Findings mapped to OWASP MASVS v2, CVSS scores assigned, remediations prioritized.
```

---

## Yaazhini — Static Analysis

### Installation & Setup

Yaazhini is a free Windows desktop APK scanner by VegaBird Technologies. The APK is loaded via the APK Scanner module, which decompiles and analyzes the application locally without sending data to external servers.

<img width="741" height="558" alt="Screenshot 2026-05-15 154345" src="https://github.com/user-attachments/assets/45b3506e-13bf-40e4-b47f-cbf52f0f61dd" />


---

### App Summary Report

After loading the APK, Yaazhini generates a summary of the application metadata before proceeding to vulnerability analysis.
<img width="1380" height="793" alt="Screenshot 2026-05-15 154732" src="https://github.com/user-attachments/assets/beb73dc0-afb6-4518-be9c-e5ab503e2b9e" />


---

### Vulnerability Report

Yaazhini identified **8 findings** across severity levels. The vulnerability tree groups findings by severity and maps each one to the affected source file.

<img width="1862" height="998" alt="Screenshot 2026-05-15 154755" src="https://github.com/user-attachments/assets/7954177e-1ced-4d8a-aa55-d12a777467a4" />

Key findings visible in the report:

- **High (1):** Insecure communication — `APICreds2Activity.java` (CVSS 8.1)
- **Medium (3):** Android debuggable enabled, Android backup vulnerability, Improper export of providers — all in `AndroidManifest.xml`

---

### Linked URLs

Yaazhini extracts all URLs hardcoded in the application source code and resolves their hosting infrastructure.

<img width="1619" height="563" alt="Screenshot 2026-05-15 154835" src="https://github.com/user-attachments/assets/0c5fcb78-1adb-4ff7-9235-8b4b41269c3b" />

| No. | Host | Server |
|-----|------|--------|
| 1 | `http://schemas.android.com` | — |
| 2 | `http://payatu.com` | Cloudflare |

Both URLs use unencrypted HTTP, consistent with FIND-001.

---

## BeVigil — External Intelligence

BeVigil is an OSINT platform that analyzes mobile applications for exposed attack surface without requiring local decompilation. The analysis target was `com.cerdillac.filmmaker` (Film Maker Pro — Movie Maker, v3.4.3), used as a complementary external analysis exercise.

### App Discovery

<img width="1919" height="729" alt="Screenshot 2026-05-15 151647" src="https://github.com/user-attachments/assets/94095bee-aa62-4e1d-8167-1021ae1c7e96" />

---

### Attack Surface Overview

BeVigil surfaces a broad set of intelligence categories from the application binary. The sidebar shows the full asset inventory extracted from `com.cerdillac.filmmaker`.

<img width="440" height="731" alt="Screenshot 2026-05-15 152605" src="https://github.com/user-attachments/assets/53842dd2-efd7-455d-a944-261c19c2f677" />

| Category | Count |
|----------|-------|
| Wordlist (endpoints) | 50 |
| Domains | 48 |
| URL Parameters | 2 |
| Email addresses | 8 |
| Filenames | 50 |
| File Paths | 50 |
| Firebase Storage Bucket | 1 |
| Firebase URL | 1 |
| IP Addresses | 31 |
| IP URLs | 4 |

---

### Wordlist — Relative Endpoints

<img width="1919" height="716" alt="Screenshot 2026-05-15 151854" src="https://github.com/user-attachments/assets/dc002543-d257-44d8-91d6-78ed6db1afbb" />

---

### Domains

48 domains were extracted from the application, including Google developer infrastructure and third-party CDN domains.

<img width="1300" height="649" alt="Screenshot 2026-05-15 152508" src="https://github.com/user-attachments/assets/61f6dbef-0b00-4f38-8930-7cb8cd00110c" />

---

### Firebase & Relative Endpoints

BeVigil detected 1 Firebase URL and 50 relative endpoints. The majority of endpoints correspond to GLSL shader asset paths (OpenGL ES resources), classified as false positives after verification.

<img width="1046" height="585" alt="Screenshot 2026-05-15 152842" src="https://github.com/user-attachments/assets/b0e544e2-e3b3-45b4-a8b2-f4e81c019dac" />

The Firebase database `bff-test.firebaseio.com` was confirmed deactivated upon manual testing:
```
{"error": "The Firebase database 'bff-test' has been deactivated."}
```
Residual risk: **low**.

---

## Top 5 Security Findings

### FIND-001 — Unencrypted HTTP Communication
- **Severity:** HIGH (CVSS 8.1)
- **Location:** `APICreds2Activity.java` — Line 23
- **Impact:** Credentials and PINs transmitted in plaintext, fully exposed to Man-in-the-Middle interception.
- **Remediation:** Replace all `http://` URLs with `https://`. Enforce TLS on all connections. Implement certificate pinning.
- **OWASP Reference:** MASVS-NETWORK-1

---

### FIND-003 — Debug Mode Enabled in Production
- **Severity:** MEDIUM (CVSS 4.9)
- **Location:** `AndroidManifest.xml` — Line 7 (`android:debuggable="true"`)
- **Impact:** Attacker can attach an ADB debugger on a physical device, inspect runtime memory, extract sensitive data, or inject arbitrary code.
- **Remediation:** Set `android:debuggable="false"`. Automate this via Gradle production build configuration.
- **OWASP Reference:** MASVS-RESILIENCE-2

---

### FIND-002 — Android Backup Enabled
- **Severity:** MEDIUM (CVSS 4.9)
- **Location:** `AndroidManifest.xml` — Line 7 (`android:allowBackup="true"`)
- **Impact:** Physical access to the device allows full extraction of internal app data via `adb backup` without requiring root.
- **Remediation:** Set `android:allowBackup="false"`. If backups are required, use the `BackupAgent` API with explicit data scope control.
- **OWASP Reference:** MASVS-STORAGE-4

---

### FIND-004 — Exported ContentProvider Without Permission
- **Severity:** MEDIUM (CVSS 4.9)
- **Location:** `AndroidManifest.xml` — Line 36 (`android:exported="true"`, no permission defined)
- **Impact:** Any installed application can read, modify, or delete data managed by this provider without authentication.
- **Remediation:** Set `android:exported="false"` for internal use. Otherwise, enforce access with a `signature`-level custom permission.
- **OWASP Reference:** MASVS-PLATFORM-2

---

### FIND-006 — Sensitive Data Stored on External Storage
- **Severity:** WARNING
- **Location:** `InsecureDataStorage4Activity.java` — Line 24 (file `.uinfo.txt` written to SD card)
- **Impact:** The file is readable and writable by any application with `READ_EXTERNAL_STORAGE` permission. User data has no protection.
- **Remediation:** Move sensitive data to internal storage (`getFilesDir()` or `getDataDir()`). If external storage is unavoidable, encrypt data with AES-256 before writing.
- **OWASP Reference:** MASVS-STORAGE-2

---

## Notable False Positives

| Finding | Tool | Justification |
|---------|------|---------------|
| Firebase URL (`bff-test.firebaseio.com`) | BeVigil | Database confirmed deactivated. No data accessible. Residual risk: low. |
| Email addresses in source code | BeVigil | Likely developer contacts or error-reporting addresses. No sensitive data confirmed exposed. |
| GLSL shader paths (`ChenXinHeng0430/...`) | BeVigil | Third-party OpenGL ES video shader assets. Not API endpoints or sensitive data. |

---

## Priority Recommendations

**1. Migrate all communications to HTTPS**  
Identify and replace all HTTP URLs in source code, starting with `APICreds2Activity.java`. Enforce a Network Security Config to block plaintext traffic at the OS level.

**2. Harden the Android Manifest**  
Disable debug mode (`android:debuggable="false"`), disable ADB backups (`android:allowBackup="false"`), and restrict the ContentProvider export with an explicit permission. All three fixes can be applied in a single build iteration.

**3. Enforce a sensitive data storage policy**  
Move user data from external to internal encrypted storage. Enable `FLAG_SECURE` on the 17 identified activities to prevent screenshots. Disable clipboard access on the 34 EditText fields handling sensitive input.

---

## Annexes

| File | Description |
|------|-------------|
| `rapport_final.md` | Full security report (French) |
| `yaazhini_notes.md` | Yaazhini raw findings — 8 findings (1 High, 3 Medium, 1 Low, 1 Warning, 2 Info) |
| `bevigil_notes.md` | BeVigil external analysis — 50 endpoints, 31 IPs, 1 Firebase URL |
| `owasp_mapping.md` | 12 findings mapped to OWASP MASVS v2 |

---

## References

- [OWASP MASVS v2](https://github.com/OWASP/owasp-masvs)
- [OWASP MASTG](https://github.com/OWASP/owasp-mastg)
- [Yaazhini](https://www.vegabird.com/yaazhini/)
- [BeVigil](https://bevigil.com/)

---

*Analysis conducted on 15-MAY-2026 — Based on Yaazhini static analysis and BeVigil external intelligence of DIVA Android.*
