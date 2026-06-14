# Project Retrospective — Project KAVACH

**IIT Roorkee × Futurense Technologies · Project KAVACH**

*Members: Megha Sharma, Vinay Kumar, Kedar Pavaskar*
---

## 1. KEEP (What worked well)
- **Hypothesis-Driven Forensics**: Structuring the network analysis around competing hypotheses (Confirm/Refute/Verdict) prevented us from jumping to conclusions too early and forced us to verify every finding using primary PCAP data.
- **Reproducible Scripting**: Documenting exact `tshark` commands and Nginx/Express code diffs ensured that any reviewer can reproduce our findings and verify our remediations in under 15 minutes.
- **Structured Findings**: Organizing Workstream B findings in standalone folders with specific curl request/response evidence kept the deliverables clean and modular.

---

## 2. STOP (What did not work well / to be stopped)
- **Mixed Capture Analysis**: Running concurrent analyses on two different PCAP captures (`2018-04-30` and `2021-05-26`) created massive confusion, inconsistent IOC tables, and redundant files. We must establish a strict agreement on a single source capture before writing any forensic code.
- **Root-level File Clutter**: Storing development files, drafts, and notes directly in the repository root made navigation difficult. We must stick to the required directory structure from day one.
- **Vague IOC Attribution**: Listing IP addresses as indicators without linking them directly to specific packet ranges or confidence scores. All IOCs must be explicitly justified.

---

## 3. START (What to do differently next time)
- **Early Architecture Mapping**: Draft the Mermaid network data flows and before/after segmentation designs before diving into deep-dive packet analysis. This provides clear guidance on what network pipes are critical.
- **Local SAST Integration**: Run Semgrep CE baseline checks immediately after standing up the Docker containers, rather than waiting until the end of the development sprint.
- **MFA and Lockout Defaulting**: Prioritize identity layer controls (like MFA and account lockout) in threat modeling, recognizing that authentication bypasses are the primary bridge between separate network and web surfaces.
