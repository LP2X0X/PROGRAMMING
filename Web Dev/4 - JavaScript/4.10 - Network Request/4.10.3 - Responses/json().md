---
tags: 
 - js
 - response
 - object
 - method
---

# 1. What `.json()` Is

`.json()` is a method available on **Response** objects returned by the **Fetch API**.  
It reads the response body **stream**, parses it as **JSON**, and returns a **Promise** that resolves to a JavaScript value.

```
response.json()
```

```ad-note
It is **asynchronous** because the body is streamed and must be read before parsing.
```

---

# 2. Return Value

`.json()` returns:

- A **Promise** that resolves to a **parsed JavaScript value**:
    
    - Object
        
    - Array
        
    - String/Number/etc. (if JSON is primitive)
        

Example:

```js
const data = await response.json();
// data is a JS object or array
```

---

# 3. When It Succeeds

`.json()` succeeds when:

- The body is valid JSON text.
    
- The body can be fully read.
    

If JSON is malformed, it **throws** (Promise rejects) with a `SyntaxError`.

---

# 4. When It Fails (Promise rejects)

| Failure Condition                | Error Type / Behavior                                    |
| -------------------------------- | -------------------------------------------------------- |
| Response is not valid JSON       | `SyntaxError`                                            |
| Body has already been consumed   | `TypeError: body used already for ...`                   |
| Network error during read        | rejection from body stream                               |
| Response is `null` or empty body | Some environments resolve to an empty object, most throw |

Note: Once `.json()` is called, the body is **locked/consumed** and cannot be read again.

---

# 5. Example Usage

## Basic

```js
const res = await fetch("/api/users");
const data = await res.json();   // ← parse JSON
console.log(data);
```

## With Error Handling

```js
async function load() {
  const res = await fetch("/api/data");

  if (!res.ok) {
    throw new Error(`HTTP ${res.status}`);
  }

  const body = await res.json();
  return body;
}
```

---

# 6. Relation to Other Body Parsing Methods

|Method|Purpose|
|---|---|
|`response.text()`|Reads body as plain text (UTF-8)|
|`response.json()`|Reads and parses JSON|
|`response.blob()`|Binary data (images, files)|
|`response.arrayBuffer()`|Low-level binary buffer|
|`response.formData()`|Parses as multipart/form-data|

In all cases, **only one body read is allowed**.

---

# 7. Internal Behavior (Conceptual)

`.json()` essentially does:

```js
return response.text().then(text => JSON.parse(text));
```

But with optimized native parsing and streaming.

---

# 8. `.json()` on Request objects (rare)

`Request` objects also have `.json()`, but that is only for reading a request body in service workers or Edge/Cloudflare Workers.