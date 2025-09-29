---
tags: 
 - js
 - request
---

A request is safe if it satisfies two conditions:

### (A) **Safe methods**

Only:

- `GET` → retrieve data
    
- `HEAD` → retrieve headers only
    
- `POST` → submit data in a traditional HTML form way
    

Why not `PUT`, `PATCH`, `DELETE`?  
Because those can change server state in ways browsers can’t assume are safe, so they trigger preflight.

---

### (B) **Safe headers**

The only headers allowed are:

- `Accept`
    
- `Accept-Language`
    
- `Content-Language`
    
- `Content-Type` **but only with values:**
    
    - `application/x-www-form-urlencoded`
        
    - `multipart/form-data`
        
    - `text/plain`
        

These match what HTML forms naturally send.

❌ Any other header → unsafe.  
Example:

- `Authorization: Bearer token123`
    
- `X-API-Key: abc123`
    
- `Content-Type: application/json`  
    All of these will force a preflight request.
    

---

## 🔹Examples

✅ **Safe (simple) request**

```js
fetch("https://api.example.com/data", {
  method: "POST",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body: "name=Long&age=25"
});
```

- Method = POST → allowed
    
- Header = Content-Type: application/x-www-form-urlencoded → allowed
    
- No custom headers → safe/simple
    

---

❌ **Unsafe (needs preflight)**

```js
fetch("https://api.example.com/data", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Long" })
});
```

- Method = PUT → not safe
    
- Header = application/json → not safe
    
- Browser will first send:
    
    ```
    OPTIONS /data
    Origin: https://yourapp.com
    Access-Control-Request-Method: PUT
    Access-Control-Request-Headers: content-type
    ```
    
- If the server replies with the right CORS headers, only then will the actual `PUT` request be sent.
    

---

## 🔹 Key Idea

- **Safe/simple requests**: browser sends them directly.
    
- **Unsafe/complex requests**: browser sends a **preflight OPTIONS** first.
    

This mechanism prevents a random malicious site from sending dangerous requests on behalf of a user without the target server’s explicit consent.

---

✅ **Summary**

- Safe/simple = `GET`, `POST`, `HEAD` + only form-like headers.
    
- Unsafe/complex = everything else (`PUT`, `PATCH`, `DELETE`, `application/json`, custom headers, tokens).
    
- Unsafe requests trigger **preflight** to protect users.
    
