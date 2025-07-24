---
tags: js, term
---

### 📦 What is an **array-like object** in JavaScript?

An **array-like object** is any object that:

1. Has **indexed elements** (e.g., `obj[0]`, `obj[1]`, etc.)
    
2. Has a **`length` property**
    
3. But **is not an actual array**
    

---

### 🧪 Example:

```js
let arrayLike = {
  0: "a",
  1: "b",
  2: "c",
  length: 3
};

console.log(arrayLike[0]);     // "a"
console.log(arrayLike.length); // 3
```

- You can access items like an array
    
- But `Array.isArray(arrayLike)` returns `false`
    

---

### ❌ Limitations

You **can’t use array methods** directly:

```js
arrayLike.push("d"); // ❌ TypeError: arrayLike.push is not a function
```

---

### ✅ Convert to real array

Use `Array.from()`:

```js
let arr = Array.from(arrayLike);
console.log(arr); // ["a", "b", "c"]
```

---

### 📍 Common array-like examples

- `arguments` object in functions
    
- DOM collections (e.g. `document.querySelectorAll`)
    
- Some manually created objects like above
    
