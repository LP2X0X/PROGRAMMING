---
tags:
  - js
  - array
  - methods
---

### 🔤 **`join()`** – Combine elements into a **string**.

```javascript
const words = ['Hello', 'World'];
console.log(words.join(' ')); // 'Hello World'
```

### 📚 **`split()`** – Create an array from a **string**.

```javascript
const message = 'one,two,three';
const parts = message.split(',');
console.log(parts); // ['one', 'two', 'three']
```

### ➕ **`concat()`** – Combine arrays or values into a **new array**.
```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const result = arr1.concat(arr2);
console.log(result); // [1, 2, 3, 4]
```

- ✅ Returns a **new array**, doesn’t modify the original.
- ✅ Can combine multiple arrays or values:

```javascript
const combined = arr1.concat(5, [6, 7]);
console.log(combined); // [1, 2, 5, 6, 7]
```

- ❗ Does **not deeply flatten** nested arrays:
```javascript
console.log([1, 2].concat([3, [4, 5]])); // [1, 2, 3, [4, 5]]
```
