### [Array methods and empty slots](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#array_methods_and_empty_slots)

- Array methods have different behaviors when encountering empty slots in [sparse arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections#sparse_arrays). In general, older methods (e.g. `forEach`) treat empty slots differently from indices that contain `undefined`.
- Methods that have special treatment for empty slots include the following: [`concat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat), [`copyWithin()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin), [`every()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every), [`filter()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter), [`flat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat), [`flatMap()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap), [`forEach()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach), [`indexOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf), [`lastIndexOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/lastIndexOf), [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map), [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce), [`reduceRight()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight), [`reverse()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse), [`slice()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice), [`some()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some), [`sort()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort), and [`splice()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice). Other methods, such as `concat`, `copyWithin`, etc., preserve empty slots when doing the copying, so in the end the array is still sparse.

```ad-note
Iteration methods such as `forEach` don't visit empty slots at all.
```

```js
const colors = ["red", "yellow", "blue"];
colors[5] = "purple";
colors.forEach((item, index) => {
  console.log(`${index}: ${item}`);
});
// Output:
// 0: red
// 1: yellow
// 2: blue
// 5: purple

colors.reverse(); // ['purple', empty × 2, 'blue', 'yellow', 'red']
```

---

In JavaScript, arrays come with many powerful and frequently used methods. Here are the most commonly used ones, grouped by functionality:

---

## 📊 **1. Adding and Removing Elements**

### ➕ **`push()`** – Add to the **end** of an array.

```javascript
const fruits = ['apple', 'banana'];
fruits.push('cherry');
console.log(fruits); // ['apple', 'banana', 'cherry']
```

### ➖ **`pop()`** – Remove the **last** element.

```javascript
const nums = [1, 2, 3];
const last = nums.pop();
console.log(last); // 3
console.log(nums); // [1, 2]
```

### ➕ **`unshift()`** – Add to the **beginning** of an array.

```javascript
const colors = ['blue', 'green'];
colors.unshift('red');
console.log(colors); // ['red', 'blue', 'green']
```

### ➖ **`shift()`** – Remove the **first** element.

```javascript
const letters = ['a', 'b', 'c'];
const first = letters.shift();
console.log(first); // 'a'
console.log(letters); // ['b', 'c']
```

---

## 🔍 **2. Finding and Searching**

### 🔢 **`indexOf()`** – Find the **first** index of a value.

```javascript
const items = ['pen', 'pencil', 'eraser'];
console.log(items.indexOf('pencil')); // 1
console.log(items.indexOf('marker')); // -1 (not found)
```

### 🔎 **`includes()`** – Check if an item **exists**.

```javascript
const animals = ['cat', 'dog', 'bird'];
console.log(animals.includes('dog')); // true
console.log(animals.includes('lion')); // false
```

### 📐 **`find()`** – Get the **first** element that meets a condition.

```javascript
const numbers = [10, 25, 30];
const found = numbers.find(num => num > 20);
console.log(found); // 25
```

### 📏 **`findIndex()`** – Get the **index** of the first match.

```javascript
const scores = [45, 67, 89];
console.log(scores.findIndex(score => score > 50)); // 1
```

---

## 🔄 **3. Iterating and Transforming**

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

---

## 🔢 **4. Modifying Arrays**

### ✂️ **`slice()`** – Extract a portion (returns a **new** array).

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

---

## 🔡 **5. Sorting and Reversing**

### 🔤 **`sort()`** – Sort alphabetically by default.

```javascript
const names = ['John', 'Alice', 'Bob'];
names.sort();
console.log(names); // ['Alice', 'Bob', 'John']
```

For **numbers**, use a compare function:

```javascript
const nums = [30, 5, 100];
nums.sort((a, b) => a - b); // Ascending
console.log(nums); // [5, 30, 100]
```

### 🔁 **`reverse()`** – Reverse the order **in place**.

```javascript
const letters = ['a', 'b', 'c'];
letters.reverse();
console.log(letters); // ['c', 'b', 'a']
```

---

## 🔗 **6. Joining and Splitting**

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

---

## 📌 **7. Checking and Testing**

### 🔘 **`every()`** – Check if **all** elements match a condition.

```javascript
const ages = [25, 30, 35];
console.log(ages.every(age => age >= 18)); // true
```

### 🔴 **`some()`** – Check if **at least one** matches.

```javascript
const scores = [45, 60, 80];
console.log(scores.some(score => score > 75)); // true
```

---

## 📢 **Most Common Use Cases**

1. **Add/Remove Elements**: `push()`, `pop()`, `shift()`, `unshift()`
2. **Transform**: `map()`, `filter()`, `reduce()`
3. **Search**: `find()`, `indexOf()`, `includes()`
4. **Sort/Reverse**: `sort()`, `reverse()`
5. **Check Conditions**: `every()`, `some()`
6. **Iterate**: `forEach()`

Would you like deeper examples, performance tips, or best practices? 🚀