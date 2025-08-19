---
tags:
  - js
  - term
  - fundamental
---

- **Truthiness** is a special type of coercion that defines how non-Boolean values are treated as [[Boolean|Booleans]]. This allows logical operators like && and ! to be used on any type of value.

---

### **❌ There are only 7 falsy values in JavaScript:**

|**Value**|**Type**|
|---|---|
|false|Boolean|
|0|Number|
|-0|Number|
|0n|BigInt|
|""|Empty string|
|null|Null|
|undefined|Undefined|
|NaN|Not a Number|

---

### **✅ Everything else is truthy**

- Non-zero numbers: 1, -5, 3.14
    
- Non-empty strings: "hello", "false", "0"
    
- Objects: {}, [], new Date()
    
- Functions: function() {}
    

---

### **🧪 Example:**

```js
if ("0") {
  console.log("truthy"); // ✅ logs "truthy" because it's a non-empty string
}

if ([]) {
  console.log("truthy"); // ✅ empty array is truthy
}

if (0) {
  console.log("falsy"); // ❌ won't run
}
```

---

### **✅ Use in JSX or conditionals:**

```jsx
{user && <p>Hello, {user.name}</p>} // Only renders if `user` is truthy
```