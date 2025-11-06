---
tags: 
 - js
 - request 
 - overview
---

## 🧭 Overview: What is a Request in JS?

A **request** in JavaScript is how your app communicates with a **server** —  
to **fetch**, **send**, or **update** data (e.g. JSON, text, images, etc).

JS does this primarily via **HTTP requests**, using either:

1. 🧩 **`XMLHttpRequest` (XHR)** – the older API
    
2. ⚡ **`fetch()`** – the modern, promise-based API
    

---

## ⚙️ 1. `fetch()` — Modern & Recommended

`fetch()` returns a **Promise**, making it easy to use with `async/await`.

### 🔹 Basic GET request

```js
fetch('https://api.example.com/data')
  .then(response => response.json())  // convert response body to JSON
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

### 🔹 Using `async/await`

```js
async function getData() {
  try {
    const res = await fetch('https://api.example.com/data');
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

### 🔹 Sending data (POST)

```js
async function sendData() {
  const response = await fetch('https://api.example.com/data', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ name: 'Long', age: 25 })
  });

  const result = await response.json();
  console.log(result);
}
```

### 🔹 Common options

|Option|Description|
|---|---|
|`method`|HTTP method: `'GET'`, `'POST'`, `'PUT'`, `'DELETE'`|
|`headers`|Set metadata like `'Content-Type'`|
|`body`|Data to send (usually `JSON.stringify()`)|
|`mode`|CORS behavior: `'cors'`, `'same-origin'`, etc|
|`credentials`|Send cookies: `'include'`, `'same-origin'`, `'omit'`|

---

## 🧱 2. `XMLHttpRequest` — Legacy API

Older way (still works, but verbose and callback-based):

```js
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data');
xhr.onload = function() {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText));
  } else {
    console.error('Error:', xhr.status);
  }
};
xhr.onerror = function() {
  console.error('Network error');
};
xhr.send();
```

🔸 Main drawback: harder to read and handle errors compared to `fetch()`.

---

## 🌐 3. Request Lifecycle (applies to both)

1. **Create request** → `fetch()` or `XMLHttpRequest()`
    
2. **Send it to the server**
    
3. **Wait for response**
    
4. **Parse the response** (`.json()`, `.text()`, `.blob()`, etc.)
    
5. **Handle success or error**
    

---

## 🧠 4. Common Request Types

|Method|Purpose|Example|
|---|---|---|
|`GET`|Retrieve data|Fetch a user list|
|`POST`|Create new data|Submit a form|
|`PUT`|Replace existing data|Update a profile|
|`PATCH`|Partially update data|Change one field|
|`DELETE`|Remove data|Delete a record|

---

## 🧩 5. Response Object (Fetch API)

A `Response` object from `fetch()` includes:

|Property|Description|
|---|---|
|`status`|HTTP status code (e.g. 200, 404)|
|`ok`|`true` if status in 200–299 range|
|`headers`|Response headers|
|`json()`, `text()`, `blob()`|Methods to read body|

Example:

```js
const res = await fetch(url);
console.log(res.status, res.ok);
const data = await res.json();
```

---

## 🧰 6. Useful Extras

- **Abort a request:**
    
    ```js
    const controller = new AbortController();
    fetch(url, { signal: controller.signal });
    controller.abort(); // cancels request
    ```
    
- **Parallel requests:**
    
    ```js
    const [users, posts] = await Promise.all([
      fetch('/users').then(r => r.json()),
      fetch('/posts').then(r => r.json())
    ]);
    ```
    

---

## ⚡ Summary Table

|Feature|`fetch()`|`XMLHttpRequest`|
|---|---|---|
|Syntax|Promise-based|Callback-based|
|Simplicity|✅ Easier|❌ Verbose|
|JSON Handling|Built-in (`.json()`)|Manual|
|CORS|Supported|Supported|
|Progress Tracking|❌ Not built-in|✅ Yes|
|Cancel / Abort|✅ via `AbortController`|✅ via `xhr.abort()`|
|Support|Modern browsers|All browsers|