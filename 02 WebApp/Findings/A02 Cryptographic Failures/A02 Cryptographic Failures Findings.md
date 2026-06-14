# Finding F-06: Plaintext Credential Transmission over HTTP

- **OWASP Category**: A02:2021-Cryptographic Failures
- **Target Component**: Juice Shop Customer Login Portal
- **Vulnerable Endpoint**: `/rest/user/login`

---

## 1. Attack Path & Exploitation Evidence

### Step 1: Intercepting Network Traffic
The attacker places themselves on the same local network as the victim or monitors upstream unencrypted traffic using a proxy tool like Burp Suite or Wireshark.

**Request Evidence (Intercepted POST Request)**:
```http
POST /rest/user/login HTTP/1.1
Host: localhost:3000
Content-Type: application/json
Content-Length: 46

{"email":"admin@admin.com","password":"admin"}
```

**Response Evidence**:
The server authenticates the user and responds with a valid JWT session token, effectively granting the attacker full access to the intercepted account.

---

## 2. Root Cause Analysis
The application's login form transmits sensitive user credentials (email and password) inside the body of an HTTP POST request without enforcing transport-layer encryption (HTTPS/TLS). Because the transport channel is insecure, any intermediary node can read the plaintext JSON payload.

---

## 3. Business Impact
Any attacker capable of sniffing the network traffic (e.g., via rogue Wi-Fi access points, ARP spoofing, or compromised routers) can trivially harvest customer and administrative passwords in transit. This leads to mass account takeovers, unauthorized transactions, and critical data breaches without requiring interaction with the application's underlying code.

---

## 4. Remediation Patch
To fix this vulnerability, the web server must be configured to enforce HTTPS across the entire application and reject plaintext HTTP connections.

**Configuration Change (Nginx/Reverse Proxy)**:
```nginx
server {
    listen 80;
    server_name portal.meridianfinserve.com;
    
    # Force redirect all HTTP traffic to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name portal.meridianfinserve.com;

    ssl_certificate /etc/nginx/ssl/portal.crt;
    ssl_certificate_key /etc/nginx/ssl/portal.key;
    
    # Enforce modern TLS versions
    ssl_protocols TLSv1.2 TLSv1.3;
}
```
