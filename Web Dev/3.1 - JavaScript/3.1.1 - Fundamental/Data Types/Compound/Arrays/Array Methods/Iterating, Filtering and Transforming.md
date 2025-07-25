---
tags: js, array, methods
---

### 🔁 **`forEach()`** – Loop through the array (no return).

```javascript
const users = ['Alice', 'Bob'];
users.forEach(user => console.log(user));
// Alice
// Bob
```

### 📊 **`map()`** – Transform **each** element (returns a new array).

```javascript
const nums = [1, 2, 3];
const squared = nums.map(n => n * n);
console.log(squared); // [1, 4, 9]
```

### 🧹 **`filter()`** – Create a **new** array with matching elements.

```javascript
const ages = [18, 25, 16, 30];
const adults = ages.filter(age => age >= 18);
console.log(adults); // [18, 25, 30]
```

### ➗ **`reduce()`** – Reduce to a **single value** (e.g., sum).

```javascript
const prices = [10, 20, 30];
const total = prices.reduce((sum, price) => sum + price, 0);
console.log(total); // 60
```

### ➗ **`reduceRight()`** – Reduce to a **single value** but from right to left.

#### 1. **Right-associative operations**

Some expressions naturally evaluate **right to left**.

```js
// Exponentiation right to left: 2^(3^2)
let arr = [2, 3, 2];
let result = arr.reduceRight((acc, val) => val ** acc);
console.log(result); // 512 → 2 ** (3 ** 2)
```

#### 2. **Building structures in reverse**

```js
let parts = ['div', 'section', 'body'];
let html = parts.reduceRight((acc, tag) => `<${tag}>${acc}</${tag}>`, 'Hello!');
console.log(html);
// <div><section><body>Hello!</body></section></div>
```

#### 3. **Parsing or processing nested structures**

If your logic requires combining or folding items **starting from the last**, `reduceRight()` is ideal.
