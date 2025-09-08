---
tags: 
 - js
 - advance
 - error
 - technique
---

## 🔹 What does “wrapping exceptions” mean?

It means: instead of letting a **low-level error** bubble up directly (like a raw `TypeError`, `SyntaxError`, `NetworkError`, etc.), you catch it and rethrow a **custom higher-level error** that is more meaningful for your application.

Example from the article:

```js
class ReadError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause; // store the original error
  }
}

function readUser(json) {
  try {
    let user = JSON.parse(json);
    if (!user.age) {
      throw new Error("No field: age");
    }
    if (!user.name) {
      throw new Error("No field: name");
    }
    return user;
  } catch (err) {
    if (err instanceof SyntaxError) {
      throw new ReadError("Syntax Error while parsing user", err);
    } else if (err.message.startsWith("No field")) {
      throw new ReadError("Validation Error", err);
    } else {
      throw err; // rethrow unknown errors
    }
  }
}
```

Here, instead of returning raw `SyntaxError` or `Error("No field: age")`, you wrap them in a **`ReadError`**.

---

## 🔹 Why should we do this?

### 1. **Abstraction: Hide implementation details**

- Suppose your code is reading a JSON file from disk.
    
- The caller of your function doesn’t care _whether it was a `SyntaxError`, a missing field, or a type mismatch_.
    
- They just want to know: **“Something went wrong while reading the user.”**
    

So instead of leaking low-level details, you expose a **consistent higher-level error**.

---

### 2. **Better error handling in upper layers**

- Upper layers of your app (UI, API handler, etc.) don’t need to handle every possible low-level error.
    
- They only catch **one type of error** (e.g. `ReadError`) and log or display a nice message.
    

```js
try {
  let user = readUser(data);
} catch (e) {
  if (e instanceof ReadError) {
    console.log("Cannot read user: " + e.message);
    console.log("Original cause: " + e.cause);
  } else {
    throw e; // rethrow unknown stuff
  }
}
```

---

### 3. **Preserve the original error (`cause`)**

- Wrapping doesn’t throw away information — you attach the **original error object** as `cause`.
    
- This means logs/debugging still show the low-level problem.
    

In modern JS, `Error` even has a standard `cause` option:

```js
throw new Error("High-level failure", { cause: err });
```

So wrapping + chaining errors is now _built-in_.

---

### 4. **Clean error taxonomy**

- You can define different **custom error classes** for each domain of your app:
    
    - `ReadError` for parsing/reading
        
    - `DatabaseError` for DB issues
        
    - `ValidationError` for bad input
        
- Makes error handling **predictable and organized**.
    

---

## ✅ When to wrap exceptions

You should wrap when:

- You’re **raising the level of abstraction** (low-level → high-level).
    
- You want to **group many possible errors** into one “category.”
    
- You need to keep the **original error cause** but don’t want to leak it directly.
    

You shouldn’t wrap if:

- You’re just passing an error along without changing abstraction level.
    

---

👉 So the main reason is: **you give callers one clean, meaningful error type, while still preserving the original error for debugging.**