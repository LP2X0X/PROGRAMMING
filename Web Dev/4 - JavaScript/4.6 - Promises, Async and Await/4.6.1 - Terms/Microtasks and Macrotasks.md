---
tags:
  - javascript
  - term
  - advance
---

## JavaScript is Single-Threaded

- JavaScript runs on **one thread** — it has a single **call stack** that executes one piece of code at a time, top to bottom.
- When a function is called, it's pushed onto the stack. When it returns, it's popped off.
- This means JavaScript **cannot do two things at once**. If something takes a long time (a network request, a timer), the entire thread blocks.

- To avoid blocking, JavaScript offloads long-running work to the **browser** (or Node.js runtime). The browser handles timers, network requests, DOM events, etc. in the background, and when the work is done, it **queues a callback** for JavaScript to pick up later.

- This is where the **event loop** and **task queues** come in — they're the mechanism that decides *when* and *in what order* those callbacks run.

---

## The Two Task Queues

- There are two separate queues that hold async callbacks waiting to run:

| Queue | Also Called | Priority |
|-------|-----------|----------|
| **Macrotask queue** | Task queue, callback queue | Lower — runs one at a time |
| **Microtask queue** | Job queue (in the spec) | Higher — fully drained before next macrotask |

- When async work completes, its callback doesn't go onto the call stack directly. It goes into **one of these two queues**, depending on what created it.
- The **event loop** is the algorithm that reads from these queues and pushes callbacks onto the call stack.

---

## Macrotasks

### What is a macrotask?

- A **macrotask** (or just "task") is a unit of work that the browser schedules for JavaScript to run. Each macrotask is a self-contained piece of work: the event loop picks up **one macrotask**, runs it to completion, and then checks the microtask queue before picking up the next one.

### What creates macrotasks?

| Source | Example |
|--------|---------|
| `setTimeout` / `setInterval` | `setTimeout(() => {}, 0)` |
| DOM events | `click`, `keydown`, `load`, `scroll` |
| I/O callbacks | `XMLHttpRequest.onload`, `readFile` callback (Node.js) |
| `MessageChannel` | `port.postMessage()` |
| `setImmediate` | Node.js only |
| Script execution | The initial `<script>` block itself is a macrotask |

### Key behavior

- The event loop processes **one macrotask per cycle** — not all of them at once.
- After each macrotask, the browser gets a chance to **render** (repaint the UI, run `requestAnimationFrame` callbacks).
- This is why `setTimeout(..., 0)` doesn't run "immediately" — it schedules a macrotask that runs in the **next** event loop cycle, after the current code and all microtasks finish.

```js
console.log("A");

setTimeout(() => {
  console.log("B — macrotask");
}, 0);

console.log("C");
```

```
A
C
B — macrotask
```

- `"B"` is a macrotask. Even with a `0ms` delay, it waits for the current synchronous code to finish and the event loop to cycle.

---

## Microtasks

### What is a microtask?

- A **microtask** is a smaller, higher-priority unit of async work. When a microtask is queued, it doesn't wait for the next event loop cycle — it runs **as soon as the call stack is empty**, but **before** the event loop moves on to the next macrotask or rendering.

- Think of it this way: a macrotask is a "meeting" on the event loop's calendar. A microtask is a "sticky note" — the event loop handles all sticky notes **immediately after finishing the current meeting**, before looking at the next meeting on the calendar.

### What creates microtasks?

| Source | Example |
|--------|---------|
| `Promise.then` / `.catch` / `.finally` | `Promise.resolve().then(fn)` |
| `async/await` | Everything after `await` becomes a microtask |
| `queueMicrotask()` | Explicit API: `queueMicrotask(fn)` |
| `MutationObserver` | Observing DOM changes |

### Key behavior

- The microtask queue is **fully drained** before the event loop moves on. Every microtask in the queue runs, and if those microtasks add more microtasks, those run too — all before the next macrotask.
- This makes microtasks faster to execute than macrotasks (no waiting for the next loop cycle), but also more dangerous if misused.

```js
console.log("A");

queueMicrotask(() => {
  console.log("B — microtask");
});

console.log("C");
```

```
A
C
B — microtask
```

- `"B"` runs after the synchronous code (`"A"`, `"C"`) but within the **same** event loop cycle.

---

## The Event Loop Algorithm

- Here's what the event loop does on every cycle, step by step:

```
1. Pick ONE macrotask from the macrotask queue (or run the current script).
2. Execute it to completion (the call stack empties).
3. Drain the ENTIRE microtask queue:
     - Run every microtask in FIFO order.
     - If a microtask adds another microtask, that one runs too.
     - Keep going until the microtask queue is completely empty.
4. Render (if needed):
     - Run requestAnimationFrame callbacks.
     - Repaint the UI.
5. Go back to step 1.
```

- The critical insight: **steps 1-3 all happen before the browser renders**. This means:
	- Microtasks **never** yield to the renderer. If you queue a thousand microtasks, the browser cannot repaint until they're all done.
	- Macrotasks **do** yield to the renderer. After each macrotask + its microtasks, the browser gets a chance to paint.

---

## Execution Order: Micro vs Macro

- This is the most important concept — microtasks **always** run before the next macrotask:

```js
setTimeout(() => console.log("1 — macrotask"), 0);

Promise.resolve().then(() => console.log("2 — microtask"));

console.log("3 — sync");
```

```
3 — sync
2 — microtask
1 — macrotask
```

### Step-by-step trace

1. The script itself is a **macrotask**. The event loop starts executing it.
2. `setTimeout` registers a callback → goes into the **macrotask queue**.
3. `Promise.resolve().then(...)` registers a callback → goes into the **microtask queue**.
4. `console.log("3 — sync")` runs immediately (synchronous).
5. The call stack is now empty. Event loop checks the **microtask queue** → finds the Promise callback → runs it → prints `"2 — microtask"`.
6. Microtask queue is empty. Event loop moves to the **next macrotask** → the `setTimeout` callback → prints `"1 — macrotask"`.

### A more complex example

```js
console.log("1");

setTimeout(() => {
  console.log("2");
  Promise.resolve().then(() => console.log("3"));
}, 0);

Promise.resolve().then(() => {
  console.log("4");
  setTimeout(() => console.log("5"), 0);
});

console.log("6");
```

```
1
6
4
2
3
5
```

Trace:

| Step | Action | Call Stack | Microtask Queue | Macrotask Queue |
|------|--------|-----------|----------------|----------------|
| 1 | Run script (macrotask) | `script` | — | — |
| 2 | `console.log("1")` | — | — | — |
| 3 | `setTimeout(cb1)` | — | — | `cb1` |
| 4 | `Promise.then(cb2)` | — | `cb2` | `cb1` |
| 5 | `console.log("6")` | — | `cb2` | `cb1` |
| 6 | Script done → drain microtasks | `cb2` | — | `cb1` |
| 7 | `cb2`: log `"4"`, schedule `setTimeout(cb3)` | — | — | `cb1`, `cb3` |
| 8 | Next macrotask → `cb1` | `cb1` | — | `cb3` |
| 9 | `cb1`: log `"2"`, schedule `Promise.then(cb4)` | — | `cb4` | `cb3` |
| 10 | Drain microtasks → `cb4`: log `"3"` | — | — | `cb3` |
| 11 | Next macrotask → `cb3`: log `"5"` | — | — | — |

---

## How Promises Use Microtasks

- When a [[Promise]] settles (resolves or rejects), its `.then` / `.catch` / `.finally` handlers are placed into the **microtask queue** — they never run synchronously, even if the Promise is already resolved:

```js
Promise.resolve("done").then(val => console.log(val));

console.log("sync");
```

```
sync
done
```

- `Promise.resolve("done")` creates an already-resolved Promise. But `.then(...)` still goes through the microtask queue — it's **never** synchronous. This is by design: it guarantees consistent execution order regardless of whether the Promise was already settled or not.

---

## How async/await Uses Microtasks

- `async/await` is syntactic sugar over Promises, so it uses microtasks the same way. Everything **after** an `await` expression becomes a `.then()` callback under the hood:

```js
async function foo() {
  console.log("1 — before await");
  await null;
  console.log("3 — after await (microtask)");
}

foo();
console.log("2 — sync");
```

```
1 — before await
2 — sync
3 — after await (microtask)
```

### What happens at `await null`

1. `await null` is equivalent to `Promise.resolve(null).then(() => { /* rest of function */ })`.
2. The code **before** `await` runs synchronously (prints `"1"`).
3. At the `await`, the function **suspends** — it returns a pending Promise and yields control.
4. `console.log("2")` runs (synchronous code continues).
5. The call stack empties → the event loop drains microtasks → the continuation after `await` runs (prints `"3"`).

### Multiple awaits = multiple microtasks

```js
async function foo() {
  console.log("A");
  await null;
  console.log("B");
  await null;
  console.log("C");
}

foo();
console.log("D");
```

```
A
D
B
C
```

- Each `await` creates a new microtask for the code that follows it. `"B"` and `"C"` run in separate microtask ticks.

---

## Microtasks Can Spawn Microtasks

- Because the microtask queue is drained **completely** before moving on, microtasks that schedule more microtasks all run in the same cycle:

```js
queueMicrotask(() => {
  console.log("micro 1");
  queueMicrotask(() => {
    console.log("micro 2");
    queueMicrotask(() => {
      console.log("micro 3");
    });
  });
});

setTimeout(() => console.log("macro"), 0);

console.log("sync");
```

```
sync
micro 1
micro 2
micro 3
macro
```

- All three microtasks run before the macrotask, even though `micro 2` and `micro 3` didn't exist when the microtask queue started draining.

---

## Starvation: When Microtasks Block Everything

- Because the event loop drains **all** microtasks before moving on, a recursive microtask loop can **starve** macrotasks and **freeze** the UI:

```js
function loop() {
  Promise.resolve().then(loop);
}
loop();

// These will NEVER run:
setTimeout(() => console.log("timer"), 0);
// The browser can NEVER repaint.
```

- The microtask queue never empties → the event loop never reaches step 4 (render) or step 1 (next macrotask). The page freezes.

- This cannot happen with macrotasks — `setTimeout` recursion still lets the browser render between each callback because each iteration is a separate macrotask with a render opportunity in between.

---

## `queueMicrotask` vs `setTimeout(..., 0)`

| | `queueMicrotask(fn)` | `setTimeout(fn, 0)` |
|---|---|---|
| Queue | Microtask | Macrotask |
| Runs | Before next macrotask | In the next event loop cycle |
| Rendering | Blocks rendering until done | Allows rendering before it runs |
| Use case | Need to run after current code but before anything else | Need to defer work and let the browser breathe |

- Use `queueMicrotask` when you need something to run "as soon as possible" after the current synchronous block, but still asynchronously (e.g., batching state updates, running cleanup before the next frame).
- Use `setTimeout(..., 0)` when you want to **yield** to the browser — let it render, handle events, and then run your code.

---

## Summary

```
┌─────────────────────────────────────────────┐
│              EVENT LOOP CYCLE                │
│                                              │
│  1. Run ONE macrotask                        │
│     (script, setTimeout, DOM event, I/O)     │
│                                              │
│  2. Drain ALL microtasks                     │
│     (Promise.then, await, queueMicrotask,    │
│      MutationObserver)                       │
│     ↳ if microtasks add more → run those too │
│                                              │
│  3. Render (if needed)                       │
│     (rAF, repaint, layout)                   │
│                                              │
│  4. → back to 1                              │
└─────────────────────────────────────────────┘
```

- **Macrotasks**: coarse-grained, one per loop cycle, let the browser render between them.
- **Microtasks**: fine-grained, all drain at once, block rendering until the queue is empty.
- **Promises and async/await** schedule microtasks — their callbacks always run before any pending `setTimeout` or DOM event.
- **Recursive microtasks** can starve macrotasks and freeze the page. Recursive macrotasks cannot.
