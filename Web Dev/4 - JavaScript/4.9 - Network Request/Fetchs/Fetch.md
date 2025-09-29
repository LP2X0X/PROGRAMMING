---
tags: 
 - js
 - network
 - function
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

### ☝️Getting a response is usually a two-stage process.

1. **First, the `promise`, returned by `fetch`, resolves with an object of the built-in [[Response Object]] class as soon as the server responds with headers.**

At this stage we can check HTTP status, to see whether it is successful or not, check headers, but don’t have the body yet.

The promise rejects if the `fetch` was unable to make HTTP-request, e.g. network problems, or there’s no such site. Abnormal HTTP-statuses, such as 404 or 500 do not cause an error.

We can see HTTP-status in response properties:

- **`status`** – HTTP status code, e.g. 200.
- **`ok`** – boolean, `true` if the HTTP status code is 200-299.

For example:

```javascript
let response = await fetch(url);

if (response.ok) { // if HTTP-status is 200-299
  // get the response body (the method explained below)
  let json = await response.json();
} else {
  alert("HTTP-Error: " + response.status);
}
```

**Second, to get the response body, we need to use an additional method call.**

`Response` provides multiple promise-based methods to access the body in various formats:

- **`response.text()`** – read the response and return as text,
- **`response.json()`** – parse the response as JSON,
- **`response.formData()`** – return the response as `FormData` object,
- **`response.blob()`** – return the response as Blob (binary data with type),
- **`response.arrayBuffer()`** – return the response as ArrayBuffer (low-level representation of binary data),
- additionally, `response.body` is a [ReadableStream](https://streams.spec.whatwg.org/#rs-class) object, it allows you to read the body chunk-by-chunk, we’ll see an example later.

For instance, let’s get a JSON-object with latest commits from GitHub:

```js
let url = 'https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits';
let response = await fetch(url);

let commits = await response.json(); // read response body and parse as JSON

alert(commits[0].author.login);
```

Or, the same without `await`, using pure promises syntax:

```js
fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits')
  .then(response => response.json())
  .then(commits => alert(commits[0].author.login));
```

To get the response text, `await response.text()` instead of `.json()`:

```js
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits');

let text = await response.text(); // read response body as text

alert(text.slice(0, 80) + '...');
```

As a show-case for reading in binary format, let’s fetch and show a logo image of [“fetch” specification](https://fetch.spec.whatwg.org):

```js
let response = await fetch('/article/fetch/logo-fetch.svg');

let blob = await response.blob(); // download as Blob object

// create <img> for it
let img = document.createElement('img');
img.style = 'position:fixed;top:10px;left:10px;width:100px';
document.body.append(img);

// show it
img.src = URL.createObjectURL(blob);

setTimeout(() => { // hide after three seconds
  img.remove();
  URL.revokeObjectURL(img.src);
}, 3000);
```

````ad-important
We can choose only one body-reading method.

If we’ve already got the response with `response.text()`, then `response.json()` won’t work, as the body content has already been processed.

```javascript
let text = await response.text(); // response body consumed
let parsed = await response.json(); // fails (already consumed)
```
````

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