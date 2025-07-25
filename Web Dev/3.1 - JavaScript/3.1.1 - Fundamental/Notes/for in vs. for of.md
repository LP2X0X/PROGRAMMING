---
tags: js, distinguish
---

### **✅ for (let item of arr)**

This is **modern and correct** for iterating **elements/items** in an array:

```js
const arr = ['a', 'b', 'c'];
for (let item of arr) {
  console.log(item); // a, b, c
}
```

- It gives you **values**, not indices.
- It works with all iterable objects (like arrays, strings, Maps, Sets, etc.).
- Recommended for readability and correctness when you don’t need the index.

---

### **⚠️ for (let i in arr)**

Technically **not wrong**, but **not recommended** for arrays:

```js
const arr = ['a', 'b', 'c'];
for (let i in arr) {
  console.log(i);        // 0, 1, 2 (keys)
  console.log(arr[i]);   // a, b, c
}
```

- for...in is meant for **enumerating object properties**, not array elements.
- It iterates **keys**, including any manually added properties or inherited ones.
- **Order is not guaranteed**.
- Can lead to bugs or unexpected behavior, especially with array-like objects or modified prototypes.

---

### **✅ When to use for...in**

Only use for...in when you’re working with **plain objects**, not arrays:

```js
const obj = { name: 'Alice', age: 25 };
for (let key in obj) {
  console.log(key, obj[key]);
}
```

---

### **✅ Safer alternatives**

- for...of – best for values
- arr.forEach((value, index) => {...}) – if you want both index and value
- for (let i = 0; i < arr.length; i++) – old school, gives full control

---

### **Summary**

|**Syntax**|**Use for**|**Notes**|
|---|---|---|
|for...of|Arrays, iterables|✅ Modern, clean, recommended|
|for...in|Objects|⚠️ Avoid on arrays – not incorrect, but risky|
|forEach()|Arrays|✅ Functional style, good for callbacks|
|for loop|Anything|✅ Full control, verbose|