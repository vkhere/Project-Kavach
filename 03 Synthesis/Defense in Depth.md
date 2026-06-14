# Defense-in-Depth Proposal (Workstream C)

**Project KAVACH · Workstream C · Defense in Depth**

---

This proposal details the seven-layer security remediation architecture designed to address the vulnerabilities identified in Workstream A (Network Forensics) and Workstream B (Web Application Assessment).

---

## 1. Identity Layer
- **Control**: Enforce Multi-Factor Authentication (MFA) on all Active Directory logins, partner portals, and administrative web interfaces, combined with a strict account lockout policy (lockout after 5 failed attempts).
- **Finding Citation**: 
  - [F-04 (Weak Auth / No Lockout)](../02%20Web%20App/Findings/A07%20Identification%20and%20Authentication%20Failures/A07%20Identification%20and%20Authentication%20Failures%20Findings.md)
  - [Workstream A Attack 4 (Credential Abuse)](../01%20Network/Network%20Forensics%20&%20Architecture%20Report.md#3-evidence-mapping-mitre-attack)
- **Implementation Effort**: **Medium (M)**
- **Trade-off**: Increases login friction for employees and customers, leading to a temporary increase in helpdesk password-reset tickets.

---

## 2. Perimeter Layer
- **Control**: Deploy a Web Application Firewall (WAF) to inspect incoming HTTP/HTTPS traffic for injection patterns, and enforce strict egress filtering on perimeter firewalls to block outbound ports (80/443) from internal server subnets.
- **Finding Citation**:
  - [F-01 (SQL Injection)](../02%20Web%20App/Findings/A03%20Injection/A03%20SQL%20Injection%20Findings.md)
  - [F-02 (Stored XSS)](../02%20Web%20App/Findings/A03%20Injection/A03%20XSS%20Findings.md)
  - [Workstream A Attack 1 (C2 Beaconing)](../01%20Network/Network%20Forensics%20&%20Architecture%20Report.md#3-evidence-mapping-mitre-attack)
- **Implementation Effort**: **Medium (M)**
- **Trade-off**: WAF rule sets require active tuning and can cause false positives (blocking legitimate API transactions), while egress proxies block non-HTTP protocol updates.

---

## 3. Segmentation Layer
- **Control**: Implement network microsegmentation to isolate Active Directory Domain Controllers from general user workstations. All administrative access to Server VLANs must route through a hardened Jump Host with session recording.
- **Finding Citation**:
  - [Workstream A Attack 3 (Lateral Movement via SMB/svcctl)](../01%20Network/Network%20Forensics%20&%20Architecture%20Report.md#3-evidence-mapping-mitre-attack)
  - [Workstream A Attack 5 (LDAP Discovery)](../01%20Network/Network%20Forensics%20&%20Architecture%20Report.md#3-evidence-mapping-mitre-attack)
- **Implementation Effort**: **Large (L)**
- **Trade-off**: Increases the complexity of network administration and introduces latency when deploying new server nodes.

---

## 4. Application Layer
- **Control**: Enforce secure coding standards in the CI/CD pipeline, including mandatory database query parameterization, output sanitization/escaping, and strict validation of ownership mappings on API routes.
- **Finding Citation**:
  - [F-01 (SQL Injection)](../02%20Web%20App/Findings/A03%20Injection/A03%20SQL%20Injection%20Findings.md)
  - [F-02 (Stored XSS)](../02%20Web%20App/Findings/A03%20Injection/A03%20XSS%20Findings.md)
  - [F-03 (IDOR)](../02%20Web%20App/Findings/A01%20In-Direct%20Object%20Reference/A01%20IDOR%20Findings.md)
- **Implementation Effort**: **Small (S)**
- **Trade-off**: Slows down the initial software development lifecycle (SDLC) as developers must undergo security training and code review cycles.

---

## 5. Data Layer
- **Control**: Encrypt sensitive data (PII, cards) at rest in the database, utilize column-level hashing (with strong salts), and isolate cryptographic keys in a Hardware Security Module (HSM).
- **Finding Citation**:
  - [F-01 (SQL Injection data dump)](../02%20Web%20App/Findings/A03%20Injection/A03%20SQL%20Injection%20Findings.md)
  - [F-03 (IDOR address harvesting)](../02%20Web%20App/Findings/A01%20In-Direct%20Object%20Reference/A01%20IDOR%20Findings.md)
- **Implementation Effort**: **Large (L)**
- **Trade-off**: Introduces cryptographic overhead, slightly increasing CPU utilization on database engines, and requires strict key-management governance.

---

## 6. Observability Layer
- **Control**: Forward all web application and database logs to a centralized, write-once, read-many SIEM. Deploy Network Security Monitoring (NSM) sensors to monitor boundary and east-west traffic for anomalies.
- **Finding Citation**:
  - [Workstream A Attack 1 (No-SNI Beaconing)](../01%20Network/Network%20Forensics%20&%20Architecture%20Report.md#3-evidence-mapping-mitre-attack)
  - [F-04 (Auditing Brute Force)](../02%20Web%20App/Findings/A07%20Identification%20and%20Authentication%20Failures/A07%20Identification%20and%20Authentication%20Failures%20Findings.md)
- **Implementation Effort**: **Medium (M)**
- **Trade-off**: Significantly increases storage infrastructure costs and requires dedicated SOC staff to triage alerts and prevent alert fatigue.

---

## 7. Response Layer
- **Control**: Establish automated host isolation protocols. If a workstation or server is detected communicating with blacklisted C2 IPs or initiating anomalous SMB service creation commands, isolate the host at the switch port level automatically.
- **Finding Citation**:
  - [Workstream A Attack 3 (Lateral spread to DC completed in 24 seconds)](../01%20Network/Network%20Forensics%20&%20Architecture%20Report.md#4-architecture-proposal-before-vs-proposed)
- **Implementation Effort**: **Small (S)**
- **Trade-off**: Risk of business interruption (downtime) if a false positive triggers automated network isolation on a critical production server.
