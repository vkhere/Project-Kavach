# Prompt Engineering Log — Project KAVACH

This log tracks interactions with large language models during the Project KAVACH security assessment.

---

## Prompt 1: Wireshark / tshark syntax lookup
- **Prompt**: *"How do I use tshark to filter out all TLS handshakes that don't have an SNI field in a pcap file?"*
- **Target LLM**: Gemini 1.5 Pro
- **Response**: Suggested command: `tshark -r file.pcap -Y "tls.handshake.type == 1 && !tls.handshake.extensions_server_name" -T fields -e frame.number`
- **Result & Adjustments**: The query worked but returned too many empty rows for non-TLS packets. Adjusted filter by checking that `tls.handshake.type == 1` to target only Client Hello frames.
- **Location in Deliverables**: [`network/hypotheses.md`](../network/hypotheses.md)

---

## Prompt 2: Cobalt Strike Beaconing Analysis
- **Prompt**: *"What is the default sleep jitter configuration for Cobalt Strike and how does it look mathematically in packet timestamp gaps?"*
- **Target LLM**: Claude Sonnet 4.6
- **Response**: Explained that the sleep timer represents the mean time gap and jitter (e.g. 20% jitter on a 200s sleep means random gaps between 160s and 240s).
- **Result & Adjustments**: Helped verify that the gaps seen in the 2021 PCAP (ranging strictly between 201s and 207s) represents a low-jitter configuration, proving automated C2.
- **Location in Deliverables**: [`network/hypotheses.md`](../network/hypotheses.md)

---

## Prompt 3: PHP PDO SQLi Remediation
- **Prompt**: *"How do I modify a classic PHP query `mysqli_query` that concatenates ID parameters to use safe prepared statements with PDO?"*
- **Target LLM**: Chat GPT-5.4
- **Response**: Provided example using `$stmt = $pdo->prepare(...)` and `$stmt->execute(...)`.
- **Result & Adjustments**: Handled the translation of variables exactly to match the DVWA low-security source code structure.
- **Location in Deliverables**: [`webapp/findings/F-01-sqli/finding.md`](../webapp/findings/F-01-sqli/finding.md)

---

## Prompt 4: Express.js Routing IDOR Prevention
- **Prompt**: *"In an Express app using Sequelize ORM, how do I prevent IDOR on a path like `/api/Address/:id`?"*
- **Target LLM**: GPT-OSS 120B
- **Response**: Suggested verifying that `address.UserId === req.user.id` after fetching the record.
- **Result & Adjustments**: Incorporated the JWT token user property injection matching the standard OWASP Juice Shop router context.
- **Location in Deliverables**: [`webapp/findings/F-03-idor/finding.md`](../webapp/findings/F-03-idor/finding.md)

---

## Prompt 5: Nginx Rate Limiting Syntax
- **Prompt**: *"Write the Nginx config directives to limit requests to a specific path to 5 requests per minute with a burst allowance of 3."*
- **Target LLM**: Chat GPT-5.4
- **Response**: Provided the `limit_req_zone` and `limit_req` directive syntax.
- **Result & Adjustments**: Integrated the settings cleanly into the reverse-proxy virtual server block for DVWA.
- **Location in Deliverables**: [`webapp/findings/F-04-auth-weakness/finding.md`](../webapp/findings/F-04-auth-weakness/finding.md)

---

## Prompt 6: Stored XSS Mitigation in PHP
- **Prompt**: *"What is the correct PHP function to prevent stored HTML injection in guestbook posts? htmlspecialchars or htmlentities?"*
- **Target LLM**: Chat GPT-5.4
- **Response**: Recommended `htmlspecialchars($string, ENT_QUOTES, 'UTF-8')` as it handles quotes and encoding type safely.
- **Result & Adjustments**: Applied the suggestion to the DVWA guestbook rendering loop.
- **Location in Deliverables**: [`webapp/findings/F-02-xss-stored/finding.md`](../webapp/findings/F-02-xss-stored/finding.md)

---

## Prompt 7: PASTA vs STRIDE Threat Modeling
- **Prompt**: *"What are the main differences between STRIDE and PASTA when threat modeling an integration of active directory and customer portal APIs?"*
- **Target LLM**: Gemini 1.5 Pro
- **Response**: Explained that STRIDE is software/developer-centric (focusing on system components), while PASTA is risk-centric (focusing on business objectives).
- **Result & Adjustments**: Decided to use a STRIDE matrix for components and map cross-surface attack chains to represent risk-based pivots.
- **Location in Deliverables**: [`synthesis/threat-model.md`](../synthesis/threat-model.md)

---

## Prompt 8: Failure Mode — Confident Hallucination of CVE
- **Prompt**: *"What is the CVE number for the directory traversal vulnerability in OWASP Juice Shop's /ftp static folder?"*
- **Target LLM**: Gemini 1.5 Pro
- **Response**: Confidently stated it was `CVE-2021-12345` (a placeholder or non-existent CVE).
- **How I caught it**: Checked the NVD database for the claimed CVE and found it did not exist or applied to a different product.
- **Remediation**: Realized that the Juice Shop vulnerabilities are intentionally designed as capture-the-flag exercises and do not carry distinct CVE numbers. Removed the CVE references from the finding and instead documented the underlying code logic bypass.
- **Location in Deliverables**: [`webapp/findings/F-05-security-misconfig/finding.md`](../webapp/findings/F-05-security-misconfig/finding.md)
