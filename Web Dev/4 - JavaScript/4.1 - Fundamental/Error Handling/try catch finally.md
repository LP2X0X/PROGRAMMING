---
tags: 
 - js
 - error
 - handling
---

## 1. Basic Structure

```js
try {
  // code that might throw
} catch (err) {
  // handle error
} finally {
  // optional cleanup (runs always)
}
```

- **`try`** → executes normally until an error is thrown.
    
- **`catch`** → catches the thrown error object.
    
- **`finally`** → always runs (even after `return` or `throw`).
    

![[Pasted image 20250907200751.png|center|400]]

```ad-important
Variables are local inside `try...catch...finally`
Otherwise, if we declared `let` in `try` block, it would only be visible inside of it.
```

---

### 2. Throw Anything

You can technically throw any value:

```js
try {
  throw "Oops"; // string
} catch (err) {
  console.log(err); // "Oops"
}
```

Technically, we can use anything as an error object. That may be even a primitive, like a number or a string, but it’s better to use objects, preferably with `name` and `message` properties (to stay somewhat compatible with built-in errors).

JavaScript has many built-in constructors for standard errors: `Error`, `SyntaxError`, `ReferenceError`, `TypeError` and others. We can use them to create error objects as well.

Their syntax is:
```js
let error = new Error(message);
// or
let error = new SyntaxError(message);
let error = new ReferenceError(message);
// ...
```

For built-in errors (not for any objects, just for errors), the `name` property is exactly the name of the constructor. And `message` is taken from the argument.

```js
let error = new Error("Things happen o_O");

alert(error.name); // Error
alert(error.message); // Things happen o_O
```

⚠️ Best practice: stick with `Error` objects for consistency.

---

### 3. Catch Binding

Since ES2019, the error variable is optional:

```js
try {
  riskyOperation();
} catch {
  console.log("Something failed!");
}
```

---

### 4. Nested Try-Catch

You can nest them for different handling:

```js
try {
  try {
    JSON.parse("invalid json");
  } catch (err) {
    console.log("Inner catch");
    throw err; // rethrow
  }
} catch {
  console.log("Outer catch");
}
```

---

### 5. Async Errors

- **Synchronous errors** → caught normally.
    
- **Asynchronous errors (timers, promises)** → not caught by plain `try…catch`.
    

```js
try {
  setTimeout(() => { throw "Oops"; }, 100);
} catch {
  console.log("Not caught");
}
```

For async:

```js
async function run() {
  try {
    await Promise.reject("Async fail");
  } catch (err) {
    console.log("Caught:", err);
  }
}
run();
```

That’s because the function itself is executed later, when the engine has already left the `try...catch` construct.

To catch an exception inside a scheduled function, `try...catch` must be inside that function:

```js
setTimeout(function() {
  try {
    noSuchVariable; // try...catch handles the error!
  } catch {
    alert( "error is caught here!" );
  }
}, 1000);
```

---

### 6. `finally` and `return`

The `finally` clause works for _any_ exit from `try...catch`. That includes an explicit `return`.

In the example below, there’s a `return` in `try`. In this case, `finally` is executed just before the control returns to the outer code.

```js
function func() {

  try {
    return 1;

  } catch (err) {
    /* ... */
  } finally {
    alert( 'finally' );
  }
}

alert( func() ); // first works alert from finally, and then this one
```

In other words, the function may finish with `return` or `throw`, that doesn’t matter. The `finally` clause executes in both cases.

---

### 7. `try...finally`

The `try...finally` construct, without `catch` clause, is also useful. We apply it when we don’t want to handle errors here (let them fall through), but want to be sure that processes that we started are finalized.

```js
function func() {
  // start doing something that needs completion (like measurements)
  try {
    // ...
  } finally {
    // complete that thing even if all dies
  }
}
```

In the code above, an error inside `try` always falls out, because there’s no `catch`. But `finally` works before the execution flow leaves the function.

---

✅ **Summary**

- Use `try…catch` to handle runtime errors.
    
- `finally` ensures cleanup.
    
- Works with sync code, and with async code when using `await`.
    
- You can rethrow to let outer blocks handle errors.
    