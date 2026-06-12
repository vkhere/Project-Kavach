# Joint Threat Model (Workstream C)

**Project KAVACH · Workstream C · Joint Threat Model & Synthesis**

---

## 1. Threat Modeling Methodology
This joint threat model treats Meridian FinServe as a single connected system where the web application surface and the internal enterprise network are adjacent. It utilizes the **STRIDE** methodology to map threats and highlights multi-stage attack chains that bridge network observations with application compromises.

---

## 2. STRIDE Threat Matrix

| Threat Category | System Surface | Description | Mitigating Controls | Workstream Ref |
|---|---|---|---|---|
| **Spoofing** | Web / Identity | Attacker logs in as a legitimate customer or admin by brute-forcing weak password settings or reusing harvested AD credentials. | MFA, Rate Limiting, Active Directory Lockout integration | F-04, A.4 |
| **Tampering** | Web / Database | Attacker modifies Database queries via SQL Injection to bypass authorization. | Parameterized queries | F-01 |
| **Repudiation** | Network / App | Attacker deletes audit logs or performs actions without authentication tracking due to missing per-request authZ logs. | Centralized syslog, Read-only write-once SIEM storage | A.2, F-03 |
| **Information Disclosure** | Web / PII | Attacker accesses customer address directories via IDOR or downloads system backups from directory listings. | JWT owner mapping, block directory indexes | F-03, F-05 |
| **Denial of Service** | Network / Host | Attacker shuts down domain controller authentication services or exhausts application connections via volumetric traffic. | Egress drop rules, microsegmentation | A.3, F-04 |
| **Elevation of Privilege** | Network / AD | Attacker executes local system service creation requests (`svcctl`) to elevate workstation access to AD Domain Controller access. | Restricted RPC access, administrative Jump Hosts | A.5 |

---

## 3. Cross-Surface Attack Chains (The Synthesis)

To demonstrate how the network and web security surfaces interact, two multi-stage attack chains have been modeled:

### Attack Chain 1: Web-to-Network Pivot (Outside-In)
This threat chain details how a web application compromise leads directly to internal network lateral movement.

```
[Outside Attacker] 
      │
      ▼ (Step 1)
[Exploits Stored XSS F-02 on Customer Portal]
      │
      ▼ (Step 2)
[Steals Administrator Session Cookie]
      │
      ▼ (Step 3)
[Gains Foothold on Internal Administrator Workstation 10.5.26.132]
      │
      ▼ (Step 4)
[Executes Trickbot Payload (Recon IP Lookup via api.ipify.org - Network Frame 818)]
      │
      ▼ (Step 5)
[Lateral Movement via SMB svcctl to Domain Controller 10.5.26.4 (Network Frame 10799)]
      │
      ▼ (Step 6)
[Active Directory Domain Compromised & C2 Heartbeat Started to 37.228.70.134]
```

- **Downstream Network Signatures**:
  - Outbound DNS lookups for C2 reputation checking (Spamhaus).
  - Outbound no-SNI TLS Client Hellos at a strict 202-second interval.
  - RPC `svcctl` commands crossing VLAN boundaries.

---

### Attack Chain 2: Network-to-Web Pivot (Inside-Out)
This threat chain details how an internal network compromise is leveraged to pivot and compromise the web-based customer portal.

```
[Internal Network Foothold via Phishing on Workstation 10.5.26.132]
      │
      ▼ (Step 1)
[Active Cobalt Strike Beaconing to 37.228.70.134 (Network Frame 791)]
      │
      ▼ (Step 2)
[Lateral Movement to Domain Controller 10.5.26.4 via SMB (Network Frame 10799)]
      │
      ▼ (Step 3)
[Dumps AD Domain Hashes & Extracts LDAP Credentials (Network Frame 6818)]
      │
      ▼ (Step 4)
[SSO Integration Abuse: Logs into Partner/Customer Portal as Admin using stolen AD creds]
      │
      ▼ (Step 5)
[Exploits IDOR Vulnerability F-03 on /api/Addresss to dump client PII data]
      │
      ▼ (Step 6)
[Complete Client PII Database Theft and Exfiltration]
```

- **Correlation Signal**:
  - The internal security alert of anomalous SMB traffic on the Domain Controller directly correlates with a surge in API lookup volume and IDOR attempts on the customer portal server.
