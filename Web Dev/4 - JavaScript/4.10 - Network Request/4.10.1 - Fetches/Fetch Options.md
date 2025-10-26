---
tags: 
 - js
 - fetch
 - option
---

The `fetch()` API takes two arguments:

```js
fetch(url, options)
```

- `url`: The resource you want to fetch.
    
- `options`: An optional object with request settings.
    

---

## 📋 Detailed Breakdown of `fetch` Options

### 1. **method**

- HTTP request method (default is `GET`).
    

```js
method: "POST"
```

### 2. **headers**

- An object (or `Headers` instance) with request headers.
    

```js
headers: {
  "Content-Type": "application/json",
  "Authorization": "Bearer token123"
}
```

### 3. **body**

- Request body for methods like `POST`, `PUT`, `PATCH`.
    
- Can be `string`, `FormData`, `Blob`, `ArrayBuffer`, `URLSearchParams`, etc.
    

```js
body: JSON.stringify({ name: "Long" })
```

### 4. **mode**

- CORS behavior.
    
    - `"cors"` (default): allow cross-origin requests if server allows.
        
    - `"same-origin"`: only requests to same origin.
        
    - `"no-cors"`: limited cross-origin, can’t access response body.
        

```js
mode: "cors"
```

### 5. **credentials**

- Whether to send cookies/HTTP credentials.
    
    - `"omit"`: never send.
        
    - `"same-origin"` (default): only for same-origin.
        
    - `"include"`: always send.
        

```js
credentials: "include"
```

### 6. **cache**

- How the request interacts with browser cache.
    
    - `"default"`
        
    - `"no-store"` (skip cache)
        
    - `"reload"` (force re-fetch)
        
    - `"no-cache"` (check cache, but validate)
        
    - `"force-cache"` (use cache if available)
        
    - `"only-if-cached"` (use cache, fail if not present)
        

```js
cache: "no-cache"
```

### 7. **redirect**

- Redirect handling.
    
    - `"follow"` (default): follow redirects.
        
    - `"error"`: fail if redirected.
        
    - `"manual"`: expose redirect response to JavaScript.
        

```js
redirect: "follow"
```

### 8. **referrer**

- Sets the `Referer` header.
    
    - `"about:client"` (default).
        
    - A URL string.
        

```js
referrer: "https://example.com"
```

### 9. **referrerPolicy**

- Controls how much referrer info is sent.
    
    - `"no-referrer"`
        
    - `"no-referrer-when-downgrade"` (default)
        
    - `"origin"`
        
    - `"origin-when-cross-origin"`
        
    - `"strict-origin"`
        
    - `"strict-origin-when-cross-origin"`
        
    - `"unsafe-url"`
        

```js
referrerPolicy: "no-referrer"
```

### 10. **integrity**

- Subresource Integrity (SRI). Ensures response matches a cryptographic hash.
    

```js
integrity: "sha256-abcdef123456..."
```

### 11. **keepalive**

- Allows request to outlive page (e.g., analytics ping before unload).
    

```js
keepalive: true
```

### 12. **signal**

- An `AbortSignal` to cancel request with `AbortController`.
    

```js
const controller = new AbortController();
fetch(url, { signal: controller.signal });
// later
controller.abort();
```

### 13. **priority** (experimental)

- Suggests fetch priority to the browser (`"high"`, `"low"`, `"auto"`).
    

```js
priority: "high"
```

### 14. **window**

- Legacy, must be `null` in modern use.
    

```js
window: null
```

---

## ✅ Example: Full Options in Action

```js
const controller = new AbortController();

fetch("https://api.example.com/data", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token123"
  },
  body: JSON.stringify({ userId: 1 }),
  mode: "cors",
  cache: "no-cache",
  credentials: "include",
  redirect: "follow",
  referrer: "https://myapp.com",
  referrerPolicy: "strict-origin-when-cross-origin",
  integrity: "sha256-xyz...",
  keepalive: true,
  signal: controller.signal,
  priority: "high",
  window: null
}).then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error("Fetch error:", err));
```

---

### Reference
https://javascript.info/fetch-api