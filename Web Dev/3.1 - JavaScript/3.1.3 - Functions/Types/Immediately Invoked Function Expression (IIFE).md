---
tags: js, term, fundamental
---

- An [Immediately Invoked Function Expression (IIFE)](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) is a code pattern that directly calls a function defined as an expression. It’s commonly used to create a local scope and avoid polluting the global namespace.

---

### ✅ **Basic Syntax of an IIFE**

```javascript
(function() {
    console.log('This runs immediately!');
})();
```

✅ **Explanation**:

1. `(function() { ... })` — This defines an anonymous function.
2. `()` — Immediately invokes (executes) the function.

---

### 📌 **IIFE with Arrow Function**

```javascript
(() => {
    console.log('Arrow function IIFE');
})();
```

---

### 📌 **IIFE with Parameters**

```javascript
(function(name) {
    console.log(`Hello, ${name}!`);
})('World');
```

---

### 📌 **Named IIFE**

```javascript
(function greet() {
    console.log('Named IIFE');
})();
```

---

### 📌 **Why Use IIFE?**

1. **Avoid Global Scope Pollution**: Variables inside IIFE are not accessible outside.
2. **Encapsulation**: Useful for creating private variables and functions.
3. **Immediate Execution**: Run code immediately without waiting for external calls.

---

### 📌 **Example: Private Scope with IIFE**

```javascript
const counter = (function() {
    let count = 0; // Private variable
    return {
        increment: () => ++count,
        getCount: () => count
    };
})();

console.log(counter.increment()); // 1
console.log(counter.getCount());  // 1
```