---
tags: 
 - js
 - note
---

## 🔹 1. The executor is **always synchronous**

When you write:

```js
new Promise((resolve, reject) => {
  console.log("inside executor");
  setTimeout(() => resolve("done"), 1000);
});
console.log("after new Promise");
```

👉 Output:

```
inside executor
after new Promise
```

The executor function runs **immediately** (synchronously).  
But what you do inside it may itself schedule **asynchronous work** (`setTimeout`, `fetch`, DB call, etc.).

---

## 🔹 2. Settling happens later

The promise doesn’t “magically” wait for anything.  
It settles only when you explicitly call `resolve()` or `reject()` inside the executor.

So in:

```js
new Promise((resolve) => {
  fetch("/data.json")
    .then(resp => resp.json())
    .then(data => resolve(data));
});
```

- The executor runs **immediately**.
    
- Inside it, `fetch` starts an HTTP request (async).
    
- Later, when the request finishes, `resolve(data)` is called.
    
- Only at that moment, the Promise transitions to **fulfilled**.
    

---

## 🔹 3. `.then` waits for the promise to settle

When you attach a `.then`:

```js
p.then(result => console.log(result));
```

You’re saying: _“When this promise eventually fulfills, schedule this callback as a microtask.”_

That’s why `.then` doesn’t block — it just queues your handler.

---

## 🔹 4. Why people say “Promises wrap async work”

Because you almost always do something asynchronous inside the executor:

- `setTimeout`
    
- `fetch` / `XMLHttpRequest`
    
- reading from disk (Node.js)
    
- etc.
    

But technically nothing prevents you from doing this:

```js
new Promise(resolve => resolve(42))
  .then(console.log);
```

It still works — just resolves immediately (but handler runs on the microtask queue after sync code finishes).

---

✅ **So your statement is correct**:  
We usually put asynchronous code (like `fetch`) inside the executor, then call `resolve`/`reject` when done, and `.then` (or `await`) will “wait” (actually: schedule itself until the promise settles).