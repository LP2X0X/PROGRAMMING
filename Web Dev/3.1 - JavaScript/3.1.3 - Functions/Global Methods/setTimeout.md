---
tags: js, global, method, fundamental
---

`setTimeout()` is a **built-in JavaScript function** that schedules code to run **once after a specified delay** (in milliseconds).

```ad-note
`setTimeout` will run **only after the current call stack is empty**.
```

---

### ✅ **Syntax**

```javascript
setTimeout(function, delay, ...args)
```

- `function` – the callback to execute.
    
- `delay` – time in milliseconds before executing the callback.
    
- `...args` – optional arguments passed to the callback.
    

---

### ✅ **Example**

```javascript
setTimeout(() => {
  console.log('Hello after 2 seconds');
}, 2000);
```

This logs the message **once**, after 2 seconds.

---

### ✅ **Return value**

`setTimeout` returns a **timeout ID**:

```js
const id = setTimeout(() => {}, 1000);
clearTimeout(id); // cancels it
```

---

### ✅ **Key Facts**

- It's **asynchronous**, non-blocking.
    
- Runs **once**; use `setInterval()` for repeated execution.
    
- In browsers, it's part of the **`window` object**.
    
- In Node.js, it’s available globally via the **timers API**.
    

---

### ❌ Canceling with clearTimeout

- A call to `setTimeout` returns a “timer identifier” `timerId` that we can use to cancel the execution. The syntax to cancel:
	```js
	let timerId = setTimeout(...);
	clearTimeout(timerId);
	```

- In the code below, we schedule the function and then cancel it (changed our mind). As a result, nothing happens:
	```js
	let timerId = setTimeout(() => alert("never happens"), 1000);
	alert(timerId); // timer identifier
	
	clearTimeout(timerId);
	alert(timerId); // same identifier (doesn't become null after canceling)
	```

- As we can see from `alert` output, in a browser the timer identifier is a number. In other environments, this can be something else. For instance, Node.js returns a timer object with additional methods.

---

### ⏱️ `setTimeout` with Zero Delay – Quick Note

```js
setTimeout(() => {
  console.log('Executed after current call stack');
}, 0);
```

✅ **Key Points**:

- Even with `0` ms, the function **does not run immediately**.
- It is placed at the **end of the task queue**, so it runs **after the current code finishes**.
- This is useful for **deferring** execution or **breaking up long tasks** to avoid freezing the UI.

📌 It's a **common trick** to delay a function just enough to let the browser or JavaScript engine "breathe" between tasks.