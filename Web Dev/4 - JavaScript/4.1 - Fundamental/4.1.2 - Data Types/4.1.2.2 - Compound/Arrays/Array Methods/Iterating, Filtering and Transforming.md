---
tags:
  - js
  - array
  - methods
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
- The reduce() method of Array instances executes a user-supplied "reducer" callback function on each element of the array, in order, passing in the return value from the calculation on the preceding element. The final result of running the reducer across all elements of the array is a single value.
- The first time that the callback is run there is no "return value of the previous calculation". If supplied, an initial value may be used in its place. Otherwise the array element at index 0 is used as the initial value and iteration starts from the next element (index 1 instead of index 0).
```js
const prices = [10, 20, 30];
const total = prices.reduce((sum, price) => sum + price, 0);
console.log(total); // 60
```

```ad-note
- If no elements exist in the array:
    
    - Skip iteration.
        
    - Immediately return the initial value.
        
    
- Therefore, your callback is **never called**.
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
