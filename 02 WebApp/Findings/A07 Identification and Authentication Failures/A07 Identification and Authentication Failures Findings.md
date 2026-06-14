# Finding F-05: Identification and Authentication Failures (Weak Credentials)

- **OWASP Category**: A07:2021-Identification and Authentication Failures
- **Target Component**: Juice Shop Login Endpoint (`/rest/user/login`)
- **Vulnerable Parameter**: `email`, `password` (JSON body)

---

## 1. Attack Path & Exploitation Evidence

### Step 1: Credential Guessing / Brute Force
An attacker attempts to log in to the administrator account using easily guessable or default credentials without any rate-limiting preventing their attempts.

**Request (curl)**:
```bash
curl -i -s -X POST -H "Content-Type: application/json" \
  -d '{"email": "admin@admin.com", "password": "admin@1"}' \
  "http://localhost:3000/rest/user/login"
```

**Response Evidence**:
The application successfully authenticates the user and returns an authentication token.
```json
{
  "authentication": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "bid": 1,
    "umail": "admin@admin.com"
  }
}
```
*Result*: Attacker successfully gains full administrative access to the application using a weak password (`admin@1`).

---

## 2. Root Cause Analysis
The application permits the creation and usage of weak passwords (such as `admin@1`) that lack complexity requirements. Furthermore, the authentication endpoint does not implement adequate protections against automated credential stuffing or brute-force attacks, such as account lockouts or rate limiting.

---

## 3. Business Impact
A malicious actor can easily compromise administrative or high-privileged accounts by guessing weak passwords. This leads to complete system takeover, massive data breaches, modification of application settings, and complete loss of system integrity and confidentiality.

---

## 4. Code-Level Remediation Patch
The fix requires enforcing strict password complexity rules during account creation/update and implementing login rate limiting to prevent brute-force attacks.

```diff
--- webapp/routes/user.js.orig
+++ webapp/routes/user.js
@@ -5,6 +5,16 @@
 const security = require('../lib/insecurity')
 const utils = require('../lib/utils')
 
+// Secure Implementation: Add rate limiting to login endpoint
+const rateLimit = require("express-rate-limit");
+const loginLimiter = rateLimit({
+  windowMs: 15 * 60 * 1000, // 15 minutes
+  max: 5, // limit each IP to 5 login requests per windowMs
+  message: "Too many login attempts from this IP, please try again after 15 minutes"
+});
+
 module.exports.login = function login () {
-  return (req, res, next) => {
+  return [loginLimiter, (req, res, next) => {
     models.User.findOne({
       where: {
         email: req.body.email || '',
@@ -21,6 +31,6 @@
         }
       })
       .catch(error => {
         next(error)
       })
-  }
+  }]
 }
```
