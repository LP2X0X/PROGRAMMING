---
tags: 
 - js
 - handle
 - error
 - technique
---

## 1. What is Rethrowing?

- The “rethrowing” technique can be explained in more detail as:
	1. Catch gets all errors.
	2. In the `catch (err) {...}` block we analyze the error object `err`.
	3. If we don’t know how to handle it, we do `throw err`.


Rethrowing means **catching an error, but then throwing it again** (so it can bubble up to outer handlers).

```js
try {
  try {
    throw new Error("Something bad");
  } catch (err) {
    console.log("Inner caught:", err.message);
    throw err; // 🔁 rethrow
  }
} catch (err) {
  console.log("Outer caught:", err.message);
}
```

Output:

```
Inner caught: Something bad
Outer caught: Something bad
```

---

## 2. Why Rethrow?

- You want to **log or transform** an error, but not swallow it.
    
- You want **outer layers** (e.g., higher-level code) to still know about the error.
    

---

## 3. Selective Rethrow

Usually, we can check the error type using the `instanceof` operator:
```js
try {
  user = { /*...*/ };
} catch (err) {
  if (err instanceof ReferenceError) {
    alert('ReferenceError'); // "ReferenceError" for accessing an undefined variable
  }
}
```

You don’t always rethrow everything—maybe only unknown or critical errors:
```js
try {
  riskyOperation();
} catch (err) {
  if (err instanceof ValidationError) {
    console.log("Handled validation issue");
  } else {
    throw err; // rethrow other errors
  }
}
```

---

## 4. How Errors Bubble

- If not rethrown, the error stops in the `catch`.
    
- If rethrown, it continues up the call stack until another `try…catch` handles it, or it crashes the program.
    

---

## 5. With Async

Inside an `async` function, you can rethrow just like sync:

```js
async function fetchData() {
  try {
    await Promise.reject(new Error("Network down"));
  } catch (err) {
    console.log("Logging:", err.message);
    throw err; // rethrow
  }
}

fetchData().catch(err => console.log("Outer async catch:", err.message));
```