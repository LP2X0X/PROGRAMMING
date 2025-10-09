---
tags: 
 - js
 - request
---

An **origin** is defined by three parts together:

1. **Protocol (scheme)** → `http://` or `https://`
    
2. **Host (domain or IP)** → e.g., `example.com`, `localhost`, `192.168.1.1`
    
3. **Port** → default is `80` for HTTP, `443` for HTTPS, but custom ones matter (`:3000`, `:8080`)
    

👉 Two URLs have the same **origin** only if **all three** match.

---

## 🔹 Examples

|URL|Origin Parts|
|---|---|
|`https://example.com:443/page`|`https`, `example.com`, `443`|
|`http://example.com/page`|`http`, `example.com`, `80`|
|`https://sub.example.com/page`|`https`, `sub.example.com`, `443`|

### Same origin ✅

- `https://example.com:443/page1`
    
- `https://example.com:443/page2`
    

### Different origin ❌

- Different protocol → `http://example.com` vs `https://example.com`
    
- Different host → `https://example.com` vs `https://api.example.com`
    
- Different port → `https://example.com:443` vs `https://example.com:3000`
    

---

## 🔹 Why “Origin” Matters

- Browsers enforce the **Same-Origin Policy (SOP)** → JS code can only freely access resources (DOM, cookies, storage, etc.) from the same origin.
    
- Anything else = **cross-origin** → needs CORS or another mechanism.
    

---

✅ **So “origin” = protocol + host + port.**  
It’s the security boundary browsers use when deciding whether scripts can talk to each other.
