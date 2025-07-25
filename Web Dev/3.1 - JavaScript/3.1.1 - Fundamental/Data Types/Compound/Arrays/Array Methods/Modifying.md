---
tags: js, array, methods
---

```ad-note
JavaScript allows user to modify array while looping through it.
```

### ✂️ **`slice()`** – Extract a portion (returns a **new** array).

- It returns a new array copying to it all items from index `start` to `end` (not including `end`).
```javascript
const letters = ['a', 'b', 'c', 'd'];
console.log(letters.slice(1, 3)); // ['b', 'c']
```

### ✍️ **`splice()`** – Add, remove, or replace **in place**.

```javascript
const fruits = ['apple', 'banana', 'cherry'];
fruits.splice(1, 1, 'orange'); // Remove 1 item at index 1, add 'orange'
console.log(fruits); // ['apple', 'orange', 'cherry']
```

- The `splice` method is also able to insert the elements without any removals. For that, we need to set `deleteCount` to `0`:
```javascript
let arr = ["I", "study", "JavaScript"];

// from index 2
// delete 0
// then insert "complex" and "language"
arr.splice(2, 0, "complex", "language");

alert( arr ); // "I", "study", "complex", "language", "JavaScript"
```