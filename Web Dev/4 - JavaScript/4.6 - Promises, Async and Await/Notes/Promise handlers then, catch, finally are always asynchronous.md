---
tags: 
 - js
 - note
---

## 🔹 1. Immediate vs Deferred Execution

When you write:

```js
let p = Promise.resolve(42);

p.then(value => console.log("then:", value));

console.log("outside");
```

You might expect the `.then` handler to run immediately (since the Promise is already resolved).  
But the output is:

```
outside
then: 42
```

➡️ The `.then` handler is **always deferred**, even if the Promise is already resolved.

---

## 🔹 2. How it works under the hood

- JavaScript has two task queues:
    
    - **Macrotasks** → e.g., `setTimeout`, I/O callbacks, events.
        
    - **Microtasks** → e.g., Promise `.then`, `MutationObserver`, `queueMicrotask`.
        
- When a Promise settles (`resolve` or `reject`), its `.then`/`.catch`/`.finally` handlers are placed in the **microtask queue**.
    
- The event loop processes:
    
    1. Run one macrotask (e.g., a script, a timer).
        
    2. Empty the microtask queue (all `.then` handlers, etc.).
        
    3. Repeat.
        

That’s why `.then` handlers run **after the current synchronous code finishes**, but before the next macrotask like a `setTimeout`.

---

## 🔹 3. Example: Promise vs setTimeout

```js
Promise.resolve().then(() => console.log("promise"));
setTimeout(() => console.log("timeout"), 0);

console.log("sync");
```

Output:

```
sync
promise
timeout
```

➡️ Even though the timeout was `0ms`, Promise handlers run first because microtasks have priority over macrotasks.

---

## 🔹 4. Why always async?

If `.then` executed immediately when the Promise was already resolved:

- You’d risk unexpected reentrancy (function code resuming in the middle of itself).
    
- It would break consistency: sometimes handlers fire immediately, sometimes later.
    

By deferring them, JS guarantees:

- All `.then/.catch/.finally` callbacks are **asynchronous**, predictable.
    
- The synchronous code always finishes first.
    
- You never get into “half-executed” states.
    

---

## 🔹 5. But once they start…

Once a `.then` handler starts running, its body executes **synchronously** until completion.

```js
Promise.resolve().then(() => {
  console.log("inside then start");
  for (let i = 0; i < 1e7; i++) {} // blocks
  console.log("inside then end");
});
console.log("outside");
```

Output:

```
outside
inside then start
inside then end
```

So:

- Scheduling = async (microtask queue).
    
- Execution = sync (normal JS code, blocks like any other function).
    

---

✅ **Summary**:

- Promise handlers (`.then/.catch/.finally`) are **always asynchronous**, scheduled into the **microtask queue**.
    
- This guarantees consistent, predictable execution order.
    
- Once scheduled, their code runs synchronously until it finishes.