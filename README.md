# Project KAVACH — Two-Surface Security Assessment

## IIT Roorkee × Futurense Technologies · Confidential Engagement

Group Members: Megha Sharma, Kedar Pavaskar, Vinay Kumar

---

## 1. Engagement Charter
This repository contains the deliverables for **Project KAVACH**, an independent two-surface security assessment conducted for **Meridian FinServe Pvt. Ltd.** (a fictional Indian NBFC). The assessment treated both the network anomalies and the portal disclosures detected in the same fortnight as related, confirming a staged compromise flow.

---

## 2. Deliverable Directory Map

Following the repository guidelines, the project is structured as follows:

- **[`01 Network/`](01%20Network/) (Workstream A: Network Forensics)**
  - [`Triage-notes.md`](01%20Network/Triage-notes.md): Overview protocol hierarchy, conversations, and time bursts.
  - [`Hypothesis.md`](01%20Network/Hypothesis.md): Hypothesis deep dive for the `2021-05-26` capture (Cobalt Strike C2, exfiltration, lateral movement).
  - [`IoCs.csv`](01%20Network/IoCs.csv): Structured machine-readable file containing verified IPs, domains, and hash IOCs.
  - [`Network Forensics & Architecture Report.md`](01%20Network/Network%20Forensics%20&%20Architecture%20Report.md): Network findings summary and proposed microsegmented network architecture.
- **[`02 Web App/`](02%20Web%20App/) (Workstream B: Web Application Assessment)**
  - [`Environment/Docker-compose.yml`](02%20Web%20App/Environment/Docker-compose.yml): Reproducible test environment configurations for DVWA and OWASP Juice Shop.
  - [`Findings/`](02%20Web%20App/Findings/): Subdirectories for each identified vulnerability:
    - [`A03 SQL Injection Findings.md`](02%20Web%20App/Findings/A03%20Injection/A03%20SQL%20Injection%20Findings.md): SQL Injection login bypass.
    - [`A03 XSS Findings.md`](02%20Web%20App/Findings/A03%20Injection/A03%20XSS%20Findings.md): Stored XSS cookie stealing.
    - [`A01 IDOR Findings.md`](02%20Web%20App/Findings/A01%20In-Direct%20Object%20Reference/A01%20IDOR%20Findings.md): Insecure Direct Object Reference (IDOR) on client address book.
    - [`A07 Identification and Authentication Failures Findings.md`](02%20Web%20App/Findings/A07%20Identification%20and%20Authentication%20Failures/A07%20Identification%20and%20Authentication%20Failures%20Findings.md): Weak auth brute force / no lockout.
    - [`A04 Insecure Design Findings.md`](02%20Web%20App/Findings/A04%20-%20Insecure%20Design/A04%20Insecure%20Design%20Findings.md): Exposed Static Directory listing.
    - [`A02 Cryptographic Failures Findings.md`](02%20Web%20App/Findings/A02%20Cryptographic%20Failures/A02%20Cryptographic%20Failures%20Findings.md): Plaintext credentials transmission over HTTP.
  - [`SAST/`](02%20Web%20App/SAST/): Before and after SAST report diffs (`before.json` / `after.json`).
  - [`Web Application Security Assessment Report.md`](02%20Web%20App/Web%20Application%20Security%20Assessment%20Report.md): Final summary of the web app assessment.
- **[`03 Synthesis/`](03%20Synthesis/) (Workstream C: Joint Threat Model & Defense-in-Depth)**
  - [`Joint STRIDE Threat Model.md`](03%20Synthesis/Joint%20STRIDE%20Threat%20Model.md): Joint STRIDE threat model showing cross-surface pivots.
  - [`Defense in Depth.md`](03%20Synthesis/Defense%20in%20Depth.md): Layered controls proposal mapped to findings.
  - [`Executive Summary.md`](03%20Synthesis/Executive%20Summary.md): One page board audience summary.
- **Project Governance**
  - [`06 Project Retrospective.md`](06%20Project%20Retrospective.md): Retrospective feedback (Keep, Stop, Start).
  - [`04 Prompt Engineering Log.md`](04%20Prompt%20Engineering%20Log.md): Prompt engineering logs for LLM assistance.
  - [`05 Project Reflections.md`](05%20Project%20Reflections.md): Answers to the individual reflection questionnaire.

---

## 3. Setup and Reproducibility

### A. Network Forensics
All network analyses were conducted utilizing the public capture `2021-05-26-Trickbot-infection-with-Cobalt-Strike.pcap`. All analysis commands in the hypotheses document are fully reproducible via standard `tshark` on the command line.

### B. Web Environment
To spin up the web vulnerability testing environment:
1. Ensure Docker Desktop is installed.
2. Run:
   ```bash
   cd "02 Web App/Environment"
   docker-compose up -d
   ```
3. Access the services:
   - DVWA: `http://localhost:8080` (Database setup required on first login: admin/password)
   - Juice Shop: `http://localhost:3000`
