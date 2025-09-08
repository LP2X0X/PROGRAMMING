---
tags: 
 - js
 - error
 - term
 - object
---

### 1. **What it is**

- `Error` is a built-in constructor in JavaScript for representing **runtime errors**.
    
- All standard errors (like `TypeError`, `ReferenceError`, `RangeError`) inherit from `Error`.
    

```js
// The "pseudocode" for the built-in Error class defined by JavaScript itself
class Error {
  constructor(message) {
    this.message = message;
    this.name = "Error"; // (different names for different built-in error classes)
    this.stack = <call stack>; // non-standard, but most environments support it
  }
}
```

---

### 2. **Basic Syntax**

```js
const err = new Error("Something went wrong");
throw err;   // or just throw new Error("message");
```

---

### 3. **Standard Properties**

Every `Error` object has these common properties:

- **`name`** → The type of error (default `"Error"`, but overridden by subclasses like `"TypeError"`).
    
- **`message`** → The human-readable description of the error.
    
- **`stack`** (non-standard but widely supported) → Call stack at the time the error was created (useful for debugging).
    

Example:

```js
try {
  throw new Error("Invalid input");
} catch (e) {
  console.log(e.name);    // "Error"
  console.log(e.message); // "Invalid input"
  console.log(e.stack);   // stack trace
}
```

---

### 4. **Error Subclasses**

Common built-in error types that extend `Error`:

- `TypeError` → wrong type (e.g., calling non-function).
    
- `ReferenceError` → accessing undefined variable.
    
- `RangeError` → value not in allowed range.
    
- `SyntaxError` → invalid JS syntax (usually thrown by engine).
    
- `EvalError` → issues with `eval()`.
    
- `URIError` → issues with `encodeURI`, `decodeURI`.
    

Each behaves the same but with different `name`.

---

## ✅ Tips & Best Practices

- Always provide a **clear message** when throwing errors → `throw new Error("User not found")`.
    
- Use **custom error classes** for domain-specific errors.
    
- Don’t rely on `.stack` for logic (it’s for debugging, not standard).
    
- Avoid throwing **non-Error values** (like `throw "oops"`), always use `Error` or subclass → ensures consistent handling.
    
- Consider **wrapping lower-level errors** (rethrowing with context) for better debugging.
    