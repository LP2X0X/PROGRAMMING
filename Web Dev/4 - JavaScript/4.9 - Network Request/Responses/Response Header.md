---
tags: 
 - js
 - response
 - object
 - property
---

- Headers are **metadata** sent by the server along with the HTTP response.
    
- They **don’t affect the body directly** but tell the client how to interpret it (e.g., content type, cache, security).
    
- In `fetch`, you access them via the `Headers` object:
    
    ```js
    fetch("/api/data")
      .then(res => {
        for (let [key, value] of res.headers) {
          console.log(`${key}: ${value}`);
        }
      });
    ```
    

---

## 🔹 Categories of Common Response Headers

### 1. **General Headers**

Applied to both request and response, not related to the actual body.

- `Date` → Server time when the response was sent.
    
- `Connection` → e.g., `keep-alive` or `close`.
    
- `Transfer-Encoding` → e.g., `chunked` (how body is sent).
    

---

### 2. **Response-Specific Headers**

About the response itself:

- `Server` → Info about the web server software (e.g., `nginx`, `Apache`).
    
- `Location` → Used in **redirects** (with status codes like `301`, `302`).
    
- `Allow` → Which HTTP methods are allowed for the resource.
    

---

### 3. **Entity Headers (about the body)**

Tell the client how to interpret the content:

- `Content-Type` → MIME type (e.g., `application/json`, `text/html`).
    
- `Content-Length` → Body size in bytes.
    
- `Content-Encoding` → Compression type (`gzip`, `br`, `deflate`).
    
- `Content-Language` → Natural language (e.g., `en`, `fr`).
    
- `Content-Disposition` → Suggests how to display content (`inline`, `attachment; filename="file.pdf"`).
    

---

### 4. **Caching Headers**

Control caching behavior:

- `Cache-Control` → e.g., `no-cache`, `max-age=3600`.
    
- `Expires` → Expiry date/time for caching.
    
- `ETag` → Unique identifier for a specific version of the resource.
    
- `Last-Modified` → Last modified date of the resource.
    

---

### 5. **Security Headers**

Enhance security policies:

- `Strict-Transport-Security (HSTS)` → Forces HTTPS.
    
- `Content-Security-Policy (CSP)` → Restricts scripts, styles, etc.
    
- `X-Frame-Options` → Prevents clickjacking (`DENY`, `SAMEORIGIN`).
    
- `X-Content-Type-Options` → Stops MIME type sniffing (`nosniff`).
    
- `Access-Control-Allow-Origin` → Defines CORS allowed origins.
    

---

## 🔹 Using Headers in Fetch

You can check individual headers:

```js
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then(res => {
    console.log("Content-Type:", res.headers.get("content-type"));
    console.log("Cache-Control:", res.headers.get("cache-control"));
  });
```

Or iterate over them:

```js
for (const [key, value] of res.headers.entries()) {
  console.log(`${key}: ${value}`);
}
```

---

⚡ **Important Notes**

- Some headers are **restricted in the browser** (e.g., `Set-Cookie`) for security reasons — you won’t see them via `fetch` unless certain CORS policies are in place.
    
- Headers are **case-insensitive** (`Content-Type` and `content-type` are the same).
    
- Headers often influence how you handle the response — e.g., always check `Content-Type` before deciding whether to call `res.json()` or `res.text()`.
    