---
tags: 
 - js
 - method
---

### ✅ Definition

`fetch()` is a **built-in JavaScript function** used to make HTTP requests (GET, POST, PUT, DELETE, etc.). It returns a [[Promise]] that resolves to a `Response` object.

---

### 🖋️ Syntax

```js
fetch(input, init)
```

- **`input`** → The URL you want to fetch (string or `Request` object).
    
- **`init`** → (Optional) An object containing settings like method, headers, body, etc.
    

```ad-note
Without `options`, this is a simple GET request, downloading the contents of the `url`.
```
---

### 💡 Example: Basic GET request

```js
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    return response.json(); // parse JSON body
  })
  .then(data => console.log(data))
  .catch(error => console.error("Fetch error:", error));
```

---

# 🔍 `Response` object

When the promise resolves, you get a `Response` object. Common properties/methods:

- **`response.ok`** → `true` if status is 200–299
    
- **`response.status`** → HTTP status code (200, 404, etc.)
    
- **`response.headers`** → Response headers
    
- **Body methods (async)**:
    
    - `response.text()` → raw text
        
    - `response.json()` → parse JSON
        
    - `response.blob()` → binary (e.g., images)
        
    - `response.formData()` → form data
        
    - `response.arrayBuffer()` → raw binary buffer
        

---

# 🔨 Options (`init` object)

You can configure fetch with the second argument:

```js
fetch("https://example.com/api", {
  method: "POST", // default is GET
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token"
  },
  body: JSON.stringify({ name: "Alice", age: 25 }),
  mode: "cors",   // cors | no-cors | same-origin
  credentials: "include", // include cookies
  cache: "no-cache" // cache control
})
```

---

# 🌟 Features & Details

### 1. **Promise-based**

- Always returns a promise.
    
- No callback hell like old `XMLHttpRequest`.
    

### 2. **No automatic reject on 404/500**

- Unlike XHR, `fetch` only rejects on **network errors** (e.g., no connection).
    
- You must check `response.ok`.
    

```js
if (!response.ok) {
  throw new Error(`HTTP Error ${response.status}`);
}
```

### 3. **Streaming**

- Response body can be consumed as a stream.
    
- Useful for large files.
    

### 4. **CORS**

- By default, fetch obeys **Cross-Origin Resource Sharing (CORS)** rules.
    
- You can set `mode` to control cross-origin behavior.
    

### 5. **AbortController**

- You can cancel a fetch request.
    

```js
const controller = new AbortController();
fetch("https://example.com", { signal: controller.signal });
controller.abort(); // cancel request
```

### 6. **Error handling**

- Use `.catch()` for network errors.
    
- Use `response.ok` for HTTP errors.
    

---

# ⚠️ Gotchas

1. Fetch does **not reject** on HTTP error codes like 404/500.  
    You must check `response.ok`.
    
2. Body can be **consumed once**. If you call `response.json()`, you can’t call `response.text()` after.
    
3. Default is **no cookies**.  
    You must set `credentials: "include"` to send/receive cookies.
    

---

# ✅ Best Practices

- Always check `response.ok` before parsing.
    
- Use `async/await` for readability:
    

```js
async function getData() {
  try {
    const res = await fetch("https://jsonplaceholder.typicode.com/posts/1");
    if (!res.ok) throw new Error(`Error ${res.status}`);
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error("Fetch failed:", err);
  }
}
```

- Use `AbortController` for timeouts or user cancellations.
    
- Set proper headers when sending JSON or form data.
    
- Use `try...catch` to handle both network errors and thrown errors.
    

---

# 🔧 Comparison with Axios

- `fetch` → built-in, promise-based, but minimal (manual error handling).
    
- `axios` → external library, automatic JSON parsing, automatic error handling, request cancellation.
    

---

⚡ In short:  
`fetch()` is the **modern standard way** to perform HTTP requests in JS. It’s powerful, flexible, but requires manual handling of errors and response parsing.

---

👉 Do you want me to make a **cheat sheet table of the most common fetch use cases** (GET, POST JSON, upload file, timeout)?