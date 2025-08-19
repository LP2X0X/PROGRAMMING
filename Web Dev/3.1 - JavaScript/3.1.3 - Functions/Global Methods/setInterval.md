---
tags:
  - js
  - global
  - method
  - fundamental
---

`setInterval()` is a built-in JavaScript method that **repeatedly runs a function** with a fixed **time delay** between each call.

```ad-note
- `setInterval` is also **non-blocking and asynchronous**, but:
	- It **schedules callbacks** at fixed time intervals.
	- But **those callbacks only run when the main thread is free**.
```

---

### ✅ **Syntax**

```js
setInterval(callback, delay, ...args)
```

- `callback` – The function to execute.
- `delay` – Time in milliseconds between each execution.
- `...args` – Optional arguments passed to the callback.

---

### ✅ **Example**

```js
setInterval(() => {
  console.log('Ping every 2 seconds');
}, 2000);
```

This prints `"Ping every 2 seconds"` indefinitely.

---

### ✅ **Return Value**

Returns an **interval ID** which can be used to cancel it:

```js
const id = setInterval(() => {
  console.log('Running');
}, 1000);

// To stop it later
clearInterval(id);
```

---

### ✅ **Key Notes**

- `setInterval` runs the callback **repeatedly**, every `delay` ms.
    
- Execution time of the callback **is not counted** in the delay.
    
- In browsers, `setInterval` is part of the **`window`** object.
    
- In **Node.js**, it's part of the global **timers API**.
    
- If the callback takes longer than the interval delay, executions **might stack up**.
    

---

### ❗ Common Mistake

```js
setInterval(alert("Hi"), 1000); // ❌ runs alert immediately
```

Correct usage:

```js
setInterval(() => alert("Hi"), 1000); // ✅ runs every second
```

Would you like a `setInterval` example that stops after a few times?