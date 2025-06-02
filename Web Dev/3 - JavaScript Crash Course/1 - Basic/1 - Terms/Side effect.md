---
tags: js, term, fundamental
---

### **💥 What Is a Side Effect in JavaScript?**

A **side effect** is any **observable change** in the system **outside** of a function’s scope when the function is called.

---

### **✅ Pure Function vs ❌ Function with Side Effect**

#### **✅ Pure Function (No Side Effect)**

```js
function add(a, b) {
  return a + b;
}
```

- Does **not** modify anything outside itself.
- Given same inputs → always same output.

#### **❌ Function with Side Effect**

```js
let total = 0;

function addToTotal(num) {
  total += num; // side effect: modifies external variable
}
```

- It **modifies a global variable** (total)
- The result depends on external state

---

### **🔁 Common Examples of Side Effects**

|**Action**|**Side Effect?**|
|---|---|
|Modifying global variables|✅ Yes|
|Changing DOM (e.g. document.body)|✅ Yes|
|Writing to a file or database|✅ Yes|
|Logging to console|✅ Yes|
|Fetching from an API|✅ Yes|
|Returning a value only|❌ No|

---

### **🧠 Why Avoid Side Effects?**

- Makes code harder to **debug**, **test**, and **reuse**
- Pure functions are **predictable** and **easier to reason about**

---

### **📦 In Functional Programming**

> A “pure function” = no side effects + deterministic output
