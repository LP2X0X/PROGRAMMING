---
tags:
  - react
  - term
  - advance
---

- A Promise is a proxy for a value not necessarily known when the promise is created. It allows you to associate handlers with an asynchronous action's eventual success value or failure reason. This lets asynchronous methods return values like synchronous methods: instead of immediately returning the final value, the asynchronous method returns a promise to supply the value at some point in the future.

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

### ☝️ Notes
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
- In case something goes wrong, the executor should call `reject`. That can be done with any type of argument (just like `resolve`). But it is recommended to use [[Error object]] (or objects that inherit from `Error`).

#### The `state` and `result` are internal
- The properties `state` and `result` of the Promise object are internal. We can’t directly access them. We can use the methods `.then`/`.catch`/`.finally` for that.

---

### Consumers

- A Promise object serves as a link between the executor and the consuming functions, which will receive the result or error. Consuming functions can be registered (subscribed) using the methods `.then` and `.catch`.

#### then

- The most important, fundamental one is `.then`. 
	- The first argument of `.then` is a function that runs when the promise is resolved and receives the result.
	- The second argument of `.then` is a function that runs when the promise is rejected and receives the error.
	```js
	promise.then(
	  function(result) { /* handle a successful result */ },
	  function(error) { /* handle an error */ }
	);
	```

- For instance, here’s a reaction to a successfully resolved promise:
	```js
	let promise = new Promise(function(resolve, reject) {
	  setTimeout(() => resolve("done!"), 1000);
	});
	
	// resolve runs the first function in .then
	promise.then(
	  result => alert(result), // shows "done!" after 1 second
	  error => alert(error) // doesn't run
	);
	```

- And in the case of a rejection, the second one:
	```js
	let promise = new Promise(function(resolve, reject) {
	  setTimeout(() => reject(new Error("Whoops!")), 1000);
	});
	
	// reject runs the second function in .then
	promise.then(
	  result => alert(result), // doesn't run
	  error => alert(error) // shows "Error: Whoops!" after 1 second
	);
	```

```ad-note
If we’re interested only in successful completions, then we can provide only one function argument to `.then`:
```

![[Pasted image 20250908185524.png|center|500]]

#### catch

- If we’re interested only in errors, then we can use `null` as the first argument: `.then(null, errorHandlingFunction)`. Or we can use `.catch(errorHandlingFunction)`, which is exactly the same:
	```js
	let promise = new Promise((resolve, reject) => {
	  setTimeout(() => reject(new Error("Whoops!")), 1000);
	});
	
	// .catch(f) is the same as promise.then(null, f)
	promise.catch(alert); // shows "Error: Whoops!" after 1 second
	```
- The call `.catch(f)` is a complete analog of `.then(null, f)`, it’s just a shorthand.

#### finally

- The call `.finally(f)` is similar to `.then(f, f)` in the sense that `f` runs always, when the promise is settled: be it resolve or reject.
- The idea of `finally` is to set up a handler for performing cleanup/finalizing after the previous operations are complete. E.g. stopping loading indicators, closing no longer needed connections, etc.
```js
new Promise((resolve, reject) => {
  /* do something that takes time, and then call resolve or maybe reject */
})
  // runs when the promise is settled, doesn't matter successfully or not
  .finally(() => stop loading indicator)
  // so the loading indicator is always stopped before we go on
  .then(result => show result, err => show error)
```

</br>

- There are important differences:
	1. A `finally` handler has no arguments. In `finally` we don’t know whether the promise is successful or not. That’s all right, as our task is usually to perform “general” finalizing procedures.
	    
	2. A `finally` handler “passes through” the result or error to the next suitable handler. For instance, here the result is passed through `finally` to `then`:
		```js
		new Promise((resolve, reject) => {
		  setTimeout(() => resolve("value"), 2000);
		})
		  .finally(() => alert("Promise ready")) // triggers first
		  .then(result => alert(result)); // <-- .then shows "value"
		  
		new Promise((resolve, reject) => {
		  throw new Error("error");
		})
		  .finally(() => alert("Promise ready")) // triggers first
		  .catch(err => alert(err));  // <-- .catch shows the error
		```
		
	1. A `finally` handler also shouldn’t return anything. If it does, the returned value is silently ignored. The only exception to this rule is when a `finally` handler throws an error. Then this error goes to the next handler, instead of any previous outcome.

```ad-summary
A `finally` handler doesn’t get the outcome of the previous handler (it has no arguments). This outcome is passed through instead, to the next suitable handler.
If a `finally` handler returns something, it’s ignored.
When `finally` throws an error, then the execution goes to the nearest error handler.
```
