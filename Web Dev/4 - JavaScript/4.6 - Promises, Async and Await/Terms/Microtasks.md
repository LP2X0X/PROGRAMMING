---
tags:
  - react
  - term
  - advance
---

# 🧵 Microtasks in JavaScript

## 🔹 1. What is a microtask?

- A **microtask** is a unit of work scheduled to run **after the current synchronous code**, but **before any macrotask** (like `setTimeout` or I/O events).
    
- Microtasks are stored in the **microtask queue**.
    

**Examples of things that create microtasks:**

- `Promise.then`, `Promise.catch`, `Promise.finally`
    
- `async/await` (behind the scenes, `await` uses `.then`)
    
- `queueMicrotask()` (explicit API for microtasks)
    
- `MutationObserver`
    

---

## 🔹 2. The Event Loop & Microtasks

The JavaScript event loop works like this:

1. Take one **macrotask** (script, timer, DOM event, etc.) from the macrotask queue and run it.
    
2. When it’s done, **drain the microtask queue** (run all microtasks in order, until empty).
    
3. Then continue with the next macrotask.
    

So microtasks always run:

- After sync code
    
- Before timers or other async callbacks
    

---

## 🔹 3. Microtasks vs Macrotasks

```js
setTimeout(() => console.log("macrotask"), 0);

Promise.resolve().then(() => console.log("microtask"));

console.log("sync");
```

**Output:**

```
sync
microtask
macrotask
```

👉 Even though `setTimeout` is `0ms`, the microtask (`Promise.then`) runs first.

---

## 🔹 4. Relation to Promises

- When a Promise settles (`resolve` or `reject`), its `.then/.catch/.finally` handlers are put into the **microtask queue**.
    
- This means they **never run immediately**, even if the Promise is already resolved.
    

Example:

```js
Promise.resolve("done").then(console.log);

console.log("outside");
```

Output:

```
outside
done
```

➡️ The handler is delayed into the microtask queue.

---

## 🔹 5. Async/Await & Microtasks

`async/await` is built on top of Promises → so `await` also uses microtasks.

```js
async function foo() {
  console.log("1");
  await null;
  console.log("3");
}

foo();
console.log("2");
```

Output:

```
1
2
3
```

Explanation:

- `await null` creates a resolved Promise, whose continuation (`console.log("3")`) goes into the microtask queue.
    
- That’s why `"2"` prints before `"3"`.
    

---

## 🔹 6. Gotcha: Microtasks can starve macrotasks

Since microtasks are run **until the queue is empty**, if you keep adding microtasks recursively, timers and UI updates are delayed:

```js
function loop() {
  Promise.resolve().then(loop);
}
loop();

console.log("start");
```

⚠️ This will block macrotasks like `setTimeout`, because the event loop never gets a chance to move forward.

---

✅ **Summary**

- Microtasks = small async tasks that run **before macrotasks**.
    
- Promises rely on microtasks for `.then/.catch/.finally`.
    
- `async/await` is also powered by microtasks.
    
- Too many microtasks can delay timers and UI updates.
    