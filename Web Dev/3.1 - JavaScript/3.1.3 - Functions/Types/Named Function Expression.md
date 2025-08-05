---
tags: js, term, function, advanced
---

### 🔠 Named Function Expression (NFE) in JavaScript

A **Named Function Expression (NFE)** is a function **expression** that **has a name**. It’s often used for **self-reference** (e.g., recursion) and **better debugging**.

---

### ✅ Syntax Example

```js
let sayHi = function greet(name) {
  if (name) {
    console.log(`Hello, ${name}`);
  } else {
    greet("Guest"); // ✅ recursive self-call
  }
};

sayHi();     // Hello, Guest
// greet("John"); ❌ Error: greet is not defined
```

---

### 🔍 Key Characteristics

|Aspect|Description|
|---|---|
|**Name Scope**|`greet` is **only visible inside** the function body.|
|**Debugging**|Named functions show up by name in stack traces.|
|**Recursion**|Useful for self-recursion without relying on the outer variable name (`sayHi` in the example).|
|**Not Hoisted**|Like any function expression, it’s **not hoisted**.|

---

### ⚠️ Why not just use anonymous functions?

```js
let sayHi = function(name) {
  if (name) console.log(`Hello, ${name}`);
  else sayHi("Guest"); // ⚠️ Works only if variable sayHi isn’t reassigned
};
```

If `sayHi` is reassigned later (e.g., to `null`), this will break. With NFE, the name is **fixed internally**, so it remains callable from within.

---

### 🔁 Compare: Function Declaration vs Named Function Expression

```js
// Function Declaration
function greet(name) {
  console.log(`Hello, ${name}`);
}

// Named Function Expression
let greet = function sayHi(name) {
  console.log(`Hello, ${name}`);
};
```

|Feature|Function Declaration|Named Function Expression|
|---|---|---|
|Hoisted|✅ Yes|❌ No|
|Recursion-safe name|❌ Depends on var|✅ Inner name (fixed)|
|Scope of function name|Global/function|Local to itself|

---

### 🔐 Summary

> A **Named Function Expression** is a **function expression** that includes a **name** usable **only inside its own body**, useful for recursion and debugging.