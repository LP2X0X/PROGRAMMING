---
tags: js, term, fundamental
---

### **🧠 Function Overview in JavaScript**

JavaScript functions are **blocks of reusable code** designed to perform a task.

---

### **🛠️ Function Declaration**

```js
function greet(name) {
  return `Hello, ${name}!`;
}
```

- ✅ Can be **hoisted** (used before defined)

---

### **🧾 Function Expression**

```js
const greet = function(name) {
  return `Hello, ${name}!`;
};
```

- ❌ Not hoisted
- Can be anonymous or named

---

### **⚡ Arrow Functions (ES6+)**

```Js
const greet = (name) => `Hello, ${name}!`;
```

- Shorter syntax
- No this, arguments, super, or new.target

---

### **🎯 Parameters vs Arguments**

```ad-important
The data types of function parameters in JavaScript are not fixed.
This is because JavaScript is a dynamically typed programming language, in which the types of variables and parameters can change while the program is running, as opposed to a statically typed language, in which the types of variables and parameters are determined before the program is run.
```

```js
function add(a, b) {
  return a + b; // a, b = parameters
}

add(2, 3); // 2, 3 = arguments
```

---

### **🛑 Default Parameters**

```js
function greet(name = "Guest") {
  return `Hello, ${name}`;
}
```

---

### **♻️ Rest Parameters**

```js
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}
```

---

### **📦 Returning Values**

```js
function multiply(a, b) {
  return a * b;
}
```

---

### **🧬 IIFE (Immediately Invoked Function Expression)**

```Js
(function() {
  console.log("Runs instantly!");
})();
```

---

### **🧪 Function Constructor (not common)**

```js
const add = new Function("a", "b", "return a + b");
```

---

### **🔁 Callback Functions**

Functions passed as arguments:

```js
function greetUser(callback) {
  console.log("Hi!");
  callback();
}

greetUser(() => console.log("Nice to meet you."));
```
