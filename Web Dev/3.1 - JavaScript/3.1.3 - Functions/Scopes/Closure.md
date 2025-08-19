---
tags:
  - js
  - term
---

### 🔒 What is a **Closure** in JavaScript?

- A [closure](https://en.wikipedia.org/wiki/Closure_\(computer_programming\)) is a **function that “remembers” the variables** from its **lexical scope**, even **after the outer function has finished execution**. In some languages, that’s not possible, or a function should be written in a special way to make it happen. But as explained above, in JavaScript, all functions are naturally closures.

- That is: they automatically remember where they were created using a hidden `[[Environment]]` property, and then their code can access outer variables.
  

---

### ✅ Key Concepts

- **Lexical scope** = the scope created during function definition, not execution.
- **Closures allow functions to retain access to outer variables.**
- This is possible **even when the outer function has returned**.

---

### 🧠 Why are closures important?

Closures give functions the ability to:

- **Maintain private state**
- **Create factories or counters**
- **Avoid polluting the global scope**
- Be used in **callbacks and event handlers** effectively

---

### 📦 Example 1: Basic Closure

```js
function outer() {
  let message = "Hello from outer!";
  return function inner() {
    console.log(message); // inner uses outer's variable
  };
}

let greet = outer();
greet(); // "Hello from outer!"
```

🔍 Even though `outer()` has finished running, `greet()` still has access to `message`. That’s a closure!

---

### 🔁 Example 2: Counter using Closure

```js
function makeCounter() {
  let count = 0;
  return function() {
    return count++;
  };
}

let counter = makeCounter();
console.log(counter()); // 0
console.log(counter()); // 1
```

- `count` is **remembered** by the returned function.
    
- It’s **private** to that function, not accessible from the outside.
    

---

### 🛑 Variables are not copied — they're **remembered by reference**

```js
function outer() {
  let value = 42;
  return function inner() {
    return value;
  };
}
```

The `inner()` function doesn't get a copy of `value`; it **keeps a reference** to the original `value` in `outer()`'s scope.

---

### 🧪 Where are closures commonly used?

- **setTimeout, setInterval**
- **Event listeners**
- **Functional programming**
- **Module patterns**
- **React hooks (like useState, useEffect)**
    

---

### 🔒 Summary

> A **closure** is a function bundled together with its **lexical environment**, allowing it to **access variables from outer scopes**, even after those scopes have exited.