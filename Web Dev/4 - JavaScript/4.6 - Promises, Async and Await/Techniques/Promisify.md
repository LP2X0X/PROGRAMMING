---
tags: 
 - js
 - technique
 - advance
---

## 🔹 What is Promisification?

**Promisification** is the process of converting a function that uses **callbacks** into a function that returns a **Promise**.

This makes the function easier to use with modern async/await or `.then()` chains.

---

## 🔹 Why do it?

Older Node.js-style APIs use the **error-first callback pattern**:

```js
fs.readFile("file.txt", (err, data) => {
  if (err) throw err;
  console.log(data.toString());
});
```

❌ Problems:

- Harder to chain multiple async calls.
    
- Nesting leads to "callback hell".
    

✅ Promises (and async/await) are cleaner:

```js
fs.promises.readFile("file.txt")
  .then(data => console.log(data.toString()))
  .catch(err => console.error(err));
```

---

## 🔹 Example of Promisification

### 1. Manual conversion

```js
function promisify(fn) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}
```

Usage:

```js
const fs = require("fs");
const readFilePromise = promisify(fs.readFile);

readFilePromise("file.txt", "utf8")
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

### 2. Node.js built-in helper

Node provides `util.promisify`:

```js
const util = require("util");
const fs = require("fs");

const readFilePromise = util.promisify(fs.readFile);

readFilePromise("file.txt", "utf8")
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## 🔹 Using with async/await

Once promisified, you can use async/await:

```js
async function run() {
  try {
    const data = await readFilePromise("file.txt", "utf8");
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}

run();
```

---

## ⚡ Gotchas

1. **Callback signature must be `(err, result)`**  
    Promisification assumes Node-style callbacks.  
    If the function uses a different pattern, manual adjustment is needed.
    
2. **Multiple results**  
    If a callback returns multiple values (`callback(null, a, b)`), only the first (after error) is passed into `resolve`.
    
3. **Some APIs already have `.promises`**  
    e.g., `fs.promises.readFile` means you don’t need to promisify manually.
    

---

👉 So, **promisification = wrapping old callback APIs in a Promise**, to make them work with async/await.