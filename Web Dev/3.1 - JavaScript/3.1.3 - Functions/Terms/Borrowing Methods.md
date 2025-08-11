---
tags: js, term, advance
---

### 🔄 **Borrowing Methods in JavaScript — In Detail**

**Method borrowing** means taking a method from one object and using it on another object. This is useful when the second object **doesn’t have the method**, but its **structure is compatible** with the method from the first object.

---

### 💡 Why Use Method Borrowing?

JavaScript is flexible—objects don’t need to be instances of a class to use a method from that class. As long as the structure aligns (properties and values), you can reuse logic.

---

### 🧪 Example 1: Borrowing `Array.prototype.slice`

Let’s say you have an _array-like object_ (like `arguments` or `NodeList`) and want to treat it like a real array.

```js
function list() {
  console.log(arguments); // [Arguments] { '0': 'a', '1': 'b', '2': 'c' }

  const args = Array.prototype.slice.call(arguments);
  console.log(args); // ['a', 'b', 'c']
}

list('a', 'b', 'c');
```

Here, `slice` is **borrowed** from `Array.prototype` and applied to `arguments`.

---

### 🧪 Example 2: Borrowing `hasOwnProperty`

Some objects created via `Object.create(null)` don’t inherit from `Object.prototype`, so they don’t have `hasOwnProperty`.

```js
const obj = Object.create(null);
obj.key = "value";

console.log(obj.hasOwnProperty); // undefined

// Borrow and use safely:
console.log(Object.prototype.hasOwnProperty.call(obj, "key")); // true
```

---

### 🧪 Example 3: Borrowing `push` for array-like objects

```js
const arrayLike = {
  0: "x",
  1: "y",
  length: 2
};

Array.prototype.push.call(arrayLike, "z");

console.log(arrayLike); 
// { 0: 'x', 1: 'y', 2: 'z', length: 3 }
```

---

### ⚙️ How Does It Work?

When you do:

```js
Array.prototype.slice.call(arguments)
```

You're saying:

> “Use the `slice` method, but bind `this` inside `slice` to the `arguments` object.”

This works because `slice` internally uses `this.length` and `this[index]`, which `arguments` also has.

---

### ✅ When to Use

- Working with **array-like** or **DOM collections** (`NodeList`, `HTMLCollection`)
    
- Working with **objects without a prototype**
    
- Avoiding **polyfills** by reusing existing methods
    
- Writing **utility functions** or **decorators**
    

---

### ❗ Best Practice

- Prefer **spread syntax** or **modern utilities** if available (e.g., `[...arguments]` instead of borrowing `slice`)
    
- Only borrow methods when the object structure matches the expectation of the method
    
- Avoid overusing it for readability; modern JavaScript offers clearer alternatives
    