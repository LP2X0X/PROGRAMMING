---
tags: 
 - js
 - promise
 - api
---

### 1. What it does

- Takes an array (or iterable) of promises.
    
- Waits for **all of them to settle** (either fulfilled or rejected).
    
- Returns a **new promise** that resolves with an array of results describing the outcome of each input promise.
    
- Never rejects.
    

---

### 2. Example

```js
let promises = [
  Promise.resolve(42),
  Promise.reject("fail"),
  new Promise(res => setTimeout(() => res("done"), 500))
];

Promise.allSettled(promises).then(results => console.log(results));
```

Output:

```js
[
  { status: "fulfilled", value: 42 },
  { status: "rejected", reason: "fail" },
  { status: "fulfilled", value: "done" }
]
```

---

### 3. Structure of Results

Each result object is either:

- `{ status: "fulfilled", value: <result> }`
    
- `{ status: "rejected", reason: <error> }`
    

This way, you can inspect **all outcomes**, even errors.

---

# Difference from `Promise.all`

|Feature|`Promise.all` 🚨|`Promise.allSettled` ✅|
|---|---|---|
|When it resolves|After **all promises fulfill**|After **all promises settle** (fulfill or reject)|
|When it rejects|Immediately on **first rejection**|Never rejects|
|Result format|Array of values (only if all fulfilled)|Array of `{status, value/reason}`|
|Use case|"All must succeed, otherwise fail"|"I need the outcome of each, success or fail"|

---

### 4. Side-by-side Example

```js
let tasks = [
  Promise.resolve("ok"),
  Promise.reject("oops"),
  Promise.resolve("fine")
];

Promise.all(tasks)
  .then(results => console.log("ALL:", results))
  .catch(err => console.log("ALL error:", err));

Promise.allSettled(tasks)
  .then(results => console.log("ALLSETTLED:", results));
```

Output:

```
ALL error: oops
ALLSETTLED: [
  { status: "fulfilled", value: "ok" },
  { status: "rejected", reason: "oops" },
  { status: "fulfilled", value: "fine" }
]
```

---

✅ **Summary**

- Use **`Promise.all`** when you require _all promises to succeed_, and want to fail fast if any error happens.
    
- Use **`Promise.allSettled`** when you want to know the **final status of each promise**, regardless of success or failure.
    