---
tags:
  - js
  - array
  - methods
---

### 🔢 **`indexOf()`** – Find the **first** index of a value.

```ad-note
`indexOf` uses the strict equality `===` for comparison.
```

```javascript
const items = ['pen', 'pencil', 'eraser'];
console.log(items.indexOf('pencil')); // 1
console.log(items.indexOf('marker')); // -1 (not found)
```

----

### 🔢 **`lastIndexOf()`** – Find the **first** index of a value, but from right to left.

- The method arr.lastIndexOf is the same as `indexOf`, but looks for from right to left.
```javascript
let fruits = ['Apple', 'Orange', 'Apple']

alert( fruits.indexOf('Apple') ); // 0 (first Apple)
alert( fruits.lastIndexOf('Apple') ); // 2 (last Apple)
```

----

### 🔎 **`includes()`** – Check if an item **exists**.

```javascript
const animals = ['cat', 'dog', 'bird'];
console.log(animals.includes('dog')); // true
console.log(animals.includes('lion')); // false
```

----

### 📐 **`find()`** – Get the **first** element that meets a condition.

```javascript
const numbers = [10, 25, 30];
const found = numbers.find(num => num > 20);
console.log(found); // 25
```

----

### 📏 **`findIndex()`** – Get the **index** of the first match.

```javascript
const scores = [45, 67, 89];
console.log(scores.findIndex(score => score > 50)); // 1
```

----

### 📏 **`findLastIndex()`** – Get the **index** of the first match, but from right to left.

```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"},
  {id: 4, name: "John"}
];

// Find the index of the first John
alert(users.findIndex(user => user.name == 'John')); // 0

// Find the index of the last John
alert(users.findLastIndex(user => user.name == 'John')); // 3
```
