---
tags:
  - js
  - term
  - fundamental
---

### **📚 What Does “Hoisting” Mean in JavaScript?**

**Hoisting** is JavaScript’s default behavior of **moving declarations to the top** of their scope _before code execution_.

> 🧠 In simpler terms:

> You can use some variables/functions **before they appear** in the code — because JavaScript “hoists” their declarations to the top when compiling.

```ad-note
It does **not** literally move your code to the top of the file. Instead, **before execution begins**, [[The JavaScript Engine|the JavaScript engine]] **allocates memory for variables and functions** declared in the code.

As a result, those functions and variables **already exist in memory** when execution starts, allowing the running code to access them immediately.

![[Pasted image 20260106092601.png|center|500]]
```

---

### **✅ Function Hoisting Example**

```js
sayHi(); // ✅ Works!

function sayHi() {
  console.log("Hello!");
}
```

- Function declarations are **fully hoisted** — you can call them before they appear in code.

---

### **⚠️ Variable Hoisting (with var)**

```js
console.log(name); // undefined, not error
var name = "Alice";
```

- The var name declaration is hoisted, but **not** its value.
- JS treats it like:

```js
var name;
console.log(name); // undefined
name = "Alice";
```

---

### **❌ No Hoisting with let / const**

```js
console.log(age); // ❌ ReferenceError
let age = 30;
```

- let and const are **not accessible** before they’re declared.
    
- This is due to the **Temporal Dead Zone (TDZ)** — the period from the start of the block until the declaration is reached.
    

---

### **🧪 Summary**

|**Type**|**Hoisted?**|**Usable Before Declaration?**|
|---|---|---|
|function|✅ Yes|✅ Yes|
|var|✅ Yes|⚠️ Only gets undefined|
|let / const|✅ Yes|❌ Error (TDZ)|
