# Project KAVACH — Two-Surface Security Assessment

IIT Roorkee × Futurense Technologies · Confidential Engagement

---

## 1. Engagement Charter
This repository contains the deliverables for **Project KAVACH**, an independent two-surface security assessment conducted for **Meridian FinServe Pvt. Ltd.** (a fictional Indian NBFC). The assessment treated both the network anomalies and the portal disclosures detected in the same fortnight as related, confirming a staged compromise flow.

---

## 2. Deliverable Directory Map

Following the repository guidelines, the project is structured as follows:

- **[`network/`](network/) (Workstream A: Network Forensics)**
  - [`triage-notes.md`](network/triage-notes.md): Overview protocol hierarchy, conversations, and time bursts.
  - [`hypothesis.md`](network/hypothesis.md): Hypothesis deep dive for the `2021-05-26` capture (Cobalt Strike C2, exfiltration, lateral movement).
  - [`iocs.csv`](network/iocs.csv): Structured machine-readable file containing verified IPs, domains, and hash IOCs.
  - [`report.md`](network/report.md): Network findings summary and proposed microsegmented network architecture.
- **[`webapp/`](webapp/) (Workstream B: Web Application Assessment)**
  - [`env/docker-compose.yml`](webapp/env/docker-compose.yml): Reproducible test environment configurations for DVWA and OWASP Juice Shop.
  - [`findings/`](webapp/findings/): Subdirectories for each identified vulnerability:
    - [`F-01-sqli/finding.md`](webapp/findings/F-01-sqli/finding.md): SQL Injection login bypass.
    - [`F-02-xss-stored/finding.md`](webapp/findings/F-02-xss-stored/finding.md): Stored XSS cookie stealing.
    - [`F-03-idor/finding.md`](webapp/findings/F-03-idor/finding.md): Insecure Direct Object Reference (IDOR) on client address book.
    - [`F-04-auth-weakness/finding.md`](webapp/findings/F-04-auth-weakness/finding.md): Weak auth brute force / no lockout.
    - [`F-05-security-misconfig/finding.md`](webapp/findings/F-05-security-misconfig/finding.md): Exposed Static Directory listing.
  - [`sast/`](webapp/sast/): Before and after SAST report diffs (`before.json` / `after.json`).
  - [`report.md`](webapp/report.md): Final summary of the web app assessment.
- **[`synthesis/`](synthesis/) (Workstream C: Joint Threat Model & Defense-in-Depth)**
  - [`threat-model.md`](synthesis/threat-model.md): Joint STRIDE threat model showing cross-surface pivots.
  - [`defense-in-depth.md`](synthesis/defense-in-depth.md): Layered controls proposal mapped to findings.
- **Project Governance**
  - [`retro.md`](retro.md): Retrospective feedback (Keep, Stop, Start).
  - [`prompts/analyst.md`](prompts/analyst.md): Prompt engineering logs for LLM assistance.
  - [`reflections/analyst.md`](reflections/analyst.md): Answers to the individual reflection questionnaire.

---

## 3. Setup and Reproducibility

### A. Network Forensics
All network analyses were conducted utilizing the public capture `2021-05-26-Trickbot-infection-with-Cobalt-Strike.pcap`. All analysis commands in the hypothesis document are fully reproducible via standard `tshark` on the command line.

### B. Web Environment
To spin up the web vulnerability testing environment:
1. Ensure Docker Desktop is installed.
2. Run:
   ```bash
   cd webapp/env
   docker-compose up -d
   ```
3. Access the services:
   - DVWA: `http://localhost:8080` (Database setup required on first login. Username: admin | Password: password)
   - Juice Shop: `http://localhost:3000`
