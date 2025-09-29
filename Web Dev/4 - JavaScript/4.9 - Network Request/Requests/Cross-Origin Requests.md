---
tags: 
 - js
 - request
---

### Prerequisite

[[Origin]]

---

## 1. What is a Cross-Origin Request?

Any time a script running on one origin tries to request a resource from a **different origin**, that’s a **cross-origin request**.

Example:

```js
// Running on https://myapp.com
fetch("https://api.othersite.com/data")
```

This is cross-origin because `myapp.com` ≠ `othersite.com`.

---

## 2. Why Restrictions Exist

Browsers enforce the **Same-Origin Policy (SOP)** for security.  
This prevents malicious scripts from reading sensitive data from another site (like banking info).

Without restrictions, any site could load your `https://bank.com/account` page and read private data.

---

## 3. How Browsers Handle Cross-Origin Requests

- **Simple requests** (like GET, POST with certain headers) are often allowed but restricted in what response data you can read.
    
- **Non-simple requests** trigger a **preflight request** (an `OPTIONS` request the browser sends first) to ask the server:  
    _“Do you allow this origin to access you?”_
    

---

## 4. CORS (Cross-Origin Resource Sharing)

To explicitly allow cross-origin requests, the server must respond with special headers:

Example server response:

```
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
```

- `Access-Control-Allow-Origin` tells the browser which origins can read the response.
    
- If it’s `*`, any origin can.
    

---

## 5. Example Flow

1. Your frontend:
    
    ```js
    fetch("https://api.othersite.com/data")
    ```
    
2. Browser sees it’s cross-origin → may send preflight (`OPTIONS`).
    
3. Server replies:
    
    ```
    Access-Control-Allow-Origin: https://myapp.com
    ```
    
4. If allowed → browser lets your JS code read the response.  
    If not → your JS gets a **CORS error**.
    

---

✅ **Summary**

- Same-Origin Policy blocks scripts from freely accessing other domains.
    
- Cross-Origin Requests go through **CORS**, where the server explicitly grants permission using headers.
    
- Browsers enforce it, not JavaScript itself.
    