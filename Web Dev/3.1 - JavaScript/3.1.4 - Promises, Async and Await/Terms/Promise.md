---
tags: react, term, advance
---

- A _promise_ is a special JavaScript object that links the “producing code” and the “consuming code” together. In terms of our analogy: this is the “subscription list”. The “producing code” takes whatever time it needs to produce the promised result, and the “promise” makes that result available to all of the subscribed code when it’s ready.

---

### Syntax
- The constructor syntax for a promise object is:
	```js
	let promise = new Promise(function(resolve, reject) {
	  // executor (the producing code)
	});
	```
- The function passed to `new Promise` is called the _executor_. When `new Promise` is created, the executor runs automatically. It contains the producing code which should eventually produce the result.
</br>
- Its arguments `resolve` and `reject` are callbacks provided by JavaScript itself. Our code is only inside the executor.

- When the executor obtains the result, be it soon or late, doesn’t matter, it should call one of these callbacks:
	- `resolve(value)` — if the job is finished successfully, with result `value`.
	- `reject(error)` — if an error has occurred, `error` is the error object.
- So to summarize: the executor runs automatically and attempts to perform a job. When it is finished with the attempt, it calls `resolve` if it was successful or `reject` if there was an error.

---

### Internal properties
- The `promise` object returned by the `new Promise` constructor has these internal properties:
	- `state` — initially `"pending"`, then changes to either `"fulfilled"` when `resolve` is called or `"rejected"` when `reject` is called.
	- `result` — initially `undefined`, then changes to `value` when `resolve(value)` is called or `error` when `reject(error)` is called.
- So the executor eventually moves `promise` to one of these states:
![[Pasted image 20250818205659.png|center|500]]

---

### Notes
#### There can be only a single result or an error
- The executor should call only one `resolve` or one `reject`. Any state change is final.
- All further calls of `resolve` and `reject` are ignored:
	```js
	let promise = new Promise(function(resolve, reject) {
	  resolve("done");
	
	  reject(new Error("…")); // ignored
	  setTimeout(() => resolve("…")); // ignored
	});
	```
- The idea is that a job done by the executor may have only one result or an error.
- Also, `resolve`/`reject` expect only one argument (or none) and will ignore additional arguments.

#### Reject with `Error` objects 
- In case something goes wrong, the executor should call `reject`. That can be done with any type of argument (just like `resolve`). But it is recommended to use `Error` objects (or objects that inherit from `Error`).