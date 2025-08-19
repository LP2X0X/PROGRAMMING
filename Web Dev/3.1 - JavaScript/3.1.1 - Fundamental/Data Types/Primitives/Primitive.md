---
tags:
  - js
  - term
  - fundamental
---

A **primitive** is a basic data type that is **not an object** and has no methods. Primitives are immutable, meaning their values cannot be changed once created.

---

### **The 7 JavaScript Primitive Types:**

1. **Number**
    - Represents both integer and floating-point numbers.
    - Example: 42, 3.14, -7

2. **String**
    - Sequence of characters.
    - Example: "hello", 'JavaScript'

3. **Boolean**
    - Logical value: true or false.

4. **Undefined**
    - A variable that has been declared but not assigned a value.
    - Example:
	```
	let a;
	console.log(a); // undefined
	```

    
4. **Null**
    - Represents “no value” or “empty value”. It’s a special type with a single value: null.
5. **Symbol** (introduced in ES6)
    - A unique and immutable identifier.
6. **BigInt** (introduced in ES2020)
    - For representing integers larger than Number.MAX_SAFE_INTEGER.
    - Example: 9007199254740991n

---

### **Contrast with Objects**

Objects (including arrays and functions) are **non-primitive** and can store collections of data and have methods.

### **Why is “primitive” important?**

- Primitives are compared by **value**.
- Objects are compared by **reference**.
- Primitives are immutable — you can’t change their internal value.

---

### **Example:**

```
let a = 5;          // Number (primitive)
let b = "hello";    // String (primitive)
let c = true;       // Boolean (primitive)
let d = {name: "JS"}; // Object (non-primitive)

console.log(typeof a); // "number"
console.log(typeof d); // "object"
```
