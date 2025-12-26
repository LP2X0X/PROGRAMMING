---
tags:
  - js
  - datatype
  - fundamental
---

In JavaScript, a **`Set`** is a built-in object that lets you store **unique values** of any type — primitive or object references.

```ad-important
A value cannot be added to a set if it is [[Strict Equality Operator|strictly equal]] to any of the set's elements.
```

---

### 🔹 Key Features of `Set`

| Feature             | Description                                          |
| ------------------- | ---------------------------------------------------- |
| **Uniqueness**      | Automatically removes duplicate values               |
| **Insertion Order** | Maintains the order of elements as they are inserted | 
| **Iterable**        | Can be used with `for...of`, spread syntax, etc.     |
| **Size**            | Has a `.size` property instead of `.length`          |

---

### 🔹 Basic Syntax

```js
const mySet = new Set();

mySet.add(1);
mySet.add(5);
mySet.add(1);  // Duplicate, won't be added

console.log(mySet);       // Set { 1, 5 }
console.log(mySet.size);  // 2
```

---

### 🔹 Creating a Set from an Array

```js
const numbers = [1, 2, 3, 3, 4, 2];
const uniqueNumbers = new Set(numbers);
console.log(uniqueNumbers);  // Set { 1, 2, 3, 4 }
```

---

### 🔹 Common Methods

| Method              | Description                          |
| ------------------- | ------------------------------------ |
| `add(value)`        | Adds a new element to the set        |
| `delete(value)`     | Removes an element                   |
| `has(value)`        | Returns `true` if the value exists   |
| `clear()`           | Removes all elements                 | 
| `forEach(callback)` | Executes a callback for each element |

```js
const s = new Set([1, 2, 3]);
s.has(2);        // true
s.delete(1);     // true
s.size;          // 2
s.clear();       // Set is now empty
```

---

### 🔹 Iterating a Set

```js
const mySet = new Set(["a", "b", "c"]);

for (let value of mySet) {
  console.log(value);
}

// or with spread
console.log([...mySet]);  // [ 'a', 'b', 'c' ]
```

---

### 🔹 Use Cases

- Removing duplicates from arrays
    
- Tracking unique items (e.g., user IDs, visited pages)
    
- Efficient `has()` checks (faster than arrays in large sets)
    

---

### 🔹 Convert Set ↔ Array

```js
// Set → Array
const arr = [...mySet];

// Array → Set
const newSet = new Set(arr);
```

✅ What happens here:

- The `Set` constructor automatically removes duplicates.
    
- The order of elements is preserved (in insertion order).