---
tags: 
 - js
 - note
 - how
---

`AbortController` is a **browser API** that provides a way to **cancel asynchronous operations** — most commonly `fetch()` requests, but also any other API that supports it (like `ReadableStream`, `setTimeout` wrappers, etc.).

It exposes:

- a `signal` (an `AbortSignal` object)
    
- a method `.abort()`
    

```js
const controller = new AbortController();
const signal = controller.signal;

fetch(url, { signal });

// later
controller.abort();
```

---

## ⚙️ 2️⃣ What happens under the hood

Let’s break down what actually happens internally when you use `AbortController`.

### Step 1: `AbortController` creates an internal “abort signal”

When you do:

```js
const controller = new AbortController();
```

the browser creates:

- an `AbortSignal` object (`controller.signal`)
    
- an internal flag, often conceptually like `signal.aborted = false`
    
- a list of “abort algorithms” (callbacks) that will be run when it’s aborted
    

This `signal` is like a lightweight **event emitter** — it can dispatch an `"abort"` event.

---

### Step 2: Passing the signal to a consumer (e.g., `fetch`)

When you call:

```js
fetch(url, { signal });
```

`fetch()` internally does roughly this:

```js
if (signal.aborted) {
  // Already aborted — reject immediately
  reject(new DOMException("The operation was aborted.", "AbortError"));
}

signal.addEventListener("abort", () => {
  // If abort happens later, cancel ongoing network activity
  terminateNetworkConnection();
  reject(new DOMException("The operation was aborted.", "AbortError"));
});
```

So `fetch()` subscribes to the `"abort"` event on that signal.

---

### Step 3: When you call `controller.abort()`

```js
controller.abort();
```

Inside the `AbortController` implementation:

- It sets `signal.aborted = true`
    
- It triggers the `"abort"` event on that `signal`
    
- All registered listeners (like `fetch`) are called synchronously
    

That’s how `fetch` knows it must stop.

---

### Step 4: The fetch is actually cancelled

- The browser’s network layer receives the abort instruction
    
- It cancels the pending HTTP request (if not already completed)
    
- The associated `Promise` returned by `fetch()` is **rejected** with a special `AbortError`
    

```js
try {
  await fetch(url, { signal });
} catch (err) {
  if (err.name === "AbortError") {
    console.log("Fetch cancelled!");
  }
}
```

---

## 🧠 3️⃣ The event system behind it

`AbortSignal` inherits from the DOM’s **EventTarget** interface, which means it can dispatch and listen to events like any DOM node.

That’s why you can do:

```js
signal.addEventListener("abort", handler);
```

and internally the browser just pushes your handler into a listener list — then executes all handlers synchronously when `abort()` is called.

---

## 🔌 4️⃣ Integration with multiple tasks

You can use **one `AbortSignal` for multiple async tasks**, and abort all at once.

```js
const controller = new AbortController();

fetch(url1, { signal: controller.signal });
fetch(url2, { signal: controller.signal });

// abort both
controller.abort();
```

Internally, both `fetch` calls have registered their own abort listeners to the same signal,  
so when `abort()` fires, they _all_ get notified at once.

---

## ⚙️ 5️⃣ What happens to already-completed Promises

If the operation (e.g., fetch) has already completed and you call `abort()`, nothing happens —  
because the listener code inside `fetch` checks whether it’s still pending before acting.  
The `"abort"` event still fires, but the fetch has no more work to cancel.

---

## 🧭 6️⃣ Summary Diagram (conceptually)

```
┌───────────────────┐
│ AbortController   │
│  ├─ .signal────┐  │
│  └─ .abort()──┐│  │
└───────────────┘│  │
                  ▼  │
             ┌────────────┐
             │ AbortSignal│
             │  .aborted  │ false
             │  listeners │ → [fetch, other]
             └────────────┘
                  │
        controller.abort()
                  ▼
        signal.aborted = true
        emit "abort"
                  ▼
      All listeners notified → cancel tasks
```

---

## 💡 7️⃣ Real-world use cases

|Use Case|Description|
|---|---|
|**Cancel fetch**|Stop long-running network request when user navigates away|
|**Debounce user input**|Cancel previous search request if new keystroke comes in|
|**Cancel timeouts**|Combine with `setTimeout` for cancelable delays|
|**Abort multiple tasks**|Link several async operations to one controller|

---

## ⚙️ Example

```js
async function loadData() {
  const controller = new AbortController();

  // Cancel if user clicks "Cancel" button
  document.querySelector('#cancel').onclick = () => controller.abort();

  try {
    const response = await fetch('/data', { signal: controller.signal });
    const data = await response.json();
    console.log(data);
  } catch (err) {
    if (err.name === 'AbortError') console.log('Request cancelled');
    else throw err;
  }
}
```

---

### TL;DR

|Concept|Meaning|
|---|---|
|`AbortController`|Object that controls when to cancel|
|`signal`|An event emitter that tells async tasks to stop|
|`abort()`|Sets `signal.aborted = true` and fires `"abort"` event|
|`fetch(signal)`|Listens for `"abort"` and cancels the request|
|Behind the scenes|Event-based pub/sub between controller and async consumer|

---

Would you like me to show a **mini visual code flow** (with comments showing which part runs in microtasks vs. event loop)? It helps visualize why aborts can be instant even while awaiting a `fetch()`.