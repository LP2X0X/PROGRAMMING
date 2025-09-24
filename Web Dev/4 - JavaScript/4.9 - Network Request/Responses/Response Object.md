---
tags: 
 - js
 - network
 - object
---

A `Response` object is returned by `fetch()` or can be created manually with the `Response()` constructor.

### 📌 Properties

|Property|Type|Description|
|---|---|---|
|**`type`**|`"basic"|"cors"|
|**`url`**|`string`|The final URL (after redirects).|
|**`ok`**|`boolean`|`true` if status is between 200–299.|
|**`status`**|`number`|HTTP status code (e.g., `200`, `404`, `500`).|
|**`statusText`**|`string`|Human-readable status (e.g., `"OK"`, `"Not Found"`).|
|**`headers`**|`Headers`|A `Headers` object representing the response headers.|
|**`redirected`**|`boolean`|`true` if the request was redirected.|
|**`body`**|`ReadableStream`|The response body as a stream (low-level).|
|**`bodyUsed`**|`boolean`|`true` if the body has already been consumed.|

---

### 📌 Methods (consume the body)

Each can **only be called once** because they lock the stream:

- **`response.text()`** → resolves to raw text.
    
- **`response.json()`** → parses text into a JS object.
    
- **`response.blob()`** → gives binary large object (for files, images, etc.).
    
- **`response.arrayBuffer()`** → low-level binary buffer.
    
- **`response.formData()`** → parses `multipart/form-data`.
    

---

### 📌 Methods (control / clone)

- **`response.clone()`** → makes a copy (useful if you need to read the body twice).
    
- **`response.redirect(url, status)`** → creates a new redirect response.
    
- **`response.error()`** → creates an error response.
    
- **`response.headers.get(name)`** → get a specific header.
    

---

### 📌 Example: Inspecting Everything

```js
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => {
    console.log("Type:", response.type);
    console.log("URL:", response.url);
    console.log("Status:", response.status, response.statusText);
    console.log("OK:", response.ok);
    console.log("Redirected:", response.redirected);

    // Loop through headers
    for (let [key, value] of response.headers) {
      console.log(`Header: ${key} = ${value}`);
    }

    return response.json();
  })
  .then((data) => {
    console.log("Body as JSON:", data);
  })
  .catch((err) => console.error("Error:", err));
```

---

⚡ **Pro tip**:

- If you need to log the body **and** still use it later → use `response.clone()` so you can consume the clone for logging, and the original for actual logic.
    
