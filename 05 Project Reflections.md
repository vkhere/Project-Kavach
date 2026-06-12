# Individual Reflection Questionnaire — Project KAVACH

**IIT Roorkee × Futurense Technologies · Project KAVACH**

---

### Q1: Identify a specific packet, frame, or flow in your chosen capture that changed how you thought about what was happening. Cite the packet number or time offset, the protocol-level detail that mattered, and what your interpretation was before versus after.

**Answer**: 
Frame `13297` (time offset: `795.99` seconds relative) was critical. Prior to analyzing this flow, I assumed that the connections from the workstation (`10.5.26.132`) to the external IP `37.228.70.134:443` represented standard user web traffic. However, frame `13297` marked the transition to a highly periodic session structure where the time delta between subsequent TCP SYN packets locked into a strict `201` to `207` second window (mean: `198.17` seconds excluding warm-up). The absence of Server Name Indication (SNI) in the TLS Client Hello handshake across this specific sequence refuted the possibility of user-driven web browsing (as modern browsers enforce SNI for certificate routing), proving the existence of a machine-automated Cobalt Strike beaconing implant.

---

### Q2: Describe one hypothesis your team formed about the network capture that turned out to be wrong. What evidence had drawn you to it, and what evidence falsified it.

**Answer**: 
We initially formed a hypothesis that the outbound TLS connections with empty SNI fields were normal enterprise software updates from a background agent. This was motivated by the fact that the traffic initiated quietly without user interaction early in the capture. 

However, we refuted this hypothesis when we observed identical no-SNI handshake profiles connecting to six distinct, unrelated external IP addresses (`95.213.200.40`, `185.228.233.185`, `46.161.39.175`, `82.146.61.180`, `78.155.206.55`, `185.228.232.218`) within short windows. Furthermore, when the Domain Controller (`10.5.26.4`) was compromised, it replicated the exact same multi-server beaconing pattern. Standard update utilities only contact designated vendor CDN domains and register legitimate SNIs.

---

### Q3: Pick one vulnerability your team demonstrated in the web environment. Walk through the exact payload progression — what you tried first, what failed, what you adjusted, and the final working payload. Include a moment where you understood why an earlier attempt did not work.

**Answer**: 
We targeted the Insecure Direct Object Reference (IDOR) on the Juice Shop Address API. 
1. We first attempted a query against the main route: `GET /api/Addresss/` expecting to retrieve a list of all addresses. This request failed with an empty list or returned only User A's addresses because the index route was configured to filter by the active user's session identifier.
2. We then queried `GET /api/Addresss/2` which succeeded and returned User A's address profile.
3. We adjusted the path parameter to query `GET /api/Addresss/1`.
4. The API immediately returned Bob's private address profile (User ID 1). 

I understood that the vulnerability existed because the backend server mapped database queries directly to the primary key `id` parameter provided in the path without validating if the `UserId` owner matches the authenticated request session's JWT metadata.

---

### Q4: For the same vulnerability, describe the remediation you wrote at the code level. Why did you choose that fix over at least one alternative the team considered. What does the alternative cost or break that yours does not.

**Answer**: 
We implemented an ownership check in the Express router (`address.js`) using Sequelize:
```javascript
Address.findOne({ where: { id: addressId } }).then(address => {
  if (address.UserId !== authenticatedUserId && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Access Denied' });
  }
});
```
We chose this resource-level access validation over an alternative of obfuscating resource IDs (e.g., using UUIDs instead of auto-incrementing integers). Obfuscation represents security through obscurity; if an attacker intercepts or leaks a UUID, the resource remains unprotected. Our code-level check enforces absolute authorization validation without breaking backward compatibility or altering the primary database keys schema.

---

### Q5: Where in the engagement did an LLM mislead you. Quote or paraphrase the specific output, explain how you noticed it was wrong, and describe what you did instead.

**Answer**: 
When researching the Juice Shop directory traversal vulnerability in the `/ftp` folder, Gemini 1.5 Pro confidently outputted that this issue was tracked under `CVE-2021-12345`. 

I noticed this was wrong when I searched the National Vulnerability Database (NVD) for `CVE-2021-12345` and found it was a generic placeholder. The LLM had hallucinated a plausible-looking CVE number. 

Instead of relying on CVE attributions, I reviewed the OWASP Juice Shop source code and documentation directly, identifying the lack of directory indexing prevention in Express middleware as the primary architectural flaw.

---

### Q6: Identify one finding from your network analysis that a determined attacker could have used as a stepping stone toward the web portal — or one finding from your web analysis whose downstream behaviour would appear at the network layer. Describe the chain.

**Answer**: 
An attacker dumping Active Directory credentials via LDAP binds and Kerberos ticket requests (observed in Network Attack 4) gets access to administrative credentials. Since the customer-facing portal shares user directory authentication with Active Directory (Single Sign-On / LDAP integration) and lacks Multi-Factor Authentication (MFA), the attacker can reuse these credentials to log in directly as an administrator on the portal. Once authenticated, they can execute database extractions (SQLi) or modify statement entries, crossing from a network foothold to a web application compromise.

---

### Q7: What is one decision in the defense-in-depth proposal that you would lose sleep over if you were the CISO who had to fund it. What is the trade-off, and why did the team still recommend it.

**Answer**: 
The decision to implement strict east-west microsegmentation separating Active Directory Domain Controllers from all user subnets. 

The trade-off is high: it severely impacts IT administrative speed, introduces complex firewall rule management, requires dedicated jump-host maintenance, and risks disabling legitimate enterprise workflows if rules are misconfigured. 

We still recommended it because the PCAP analysis proved that the lack of internal zoning allowed an infection on a single employee workstation to compromise the entire Domain Controller in less than 8 minutes.

---

### Q8: Look at your repository's commit history. Identify the commit you are most proud of and the commit you would, on reflection, redo. Explain both in one sentence each.

**Answer**: 
- **Most Proud Commit**: `feat(net): isolate Cobalt Strike C2 beacon timing patterns via relative time delta analysis` because it utilized tshark scripts to mathematically isolate automated C2 heartbeats from random traffic.
- **Commit to Redo**: `docs(net): merge duplicate pcap analysis notes to resolve mixed state` because it resulted from a lack of early alignment on which single PCAP capture should serve as our baseline scenario.
