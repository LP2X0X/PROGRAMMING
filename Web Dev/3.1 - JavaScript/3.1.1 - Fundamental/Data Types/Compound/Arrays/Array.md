---
tags: js, datatype, fundamental
---

## 📦 What is an Array?

An **array** is a special object used to **store multiple values** in a single variable.

```js
let fruits = ["apple", "banana", "cherry"];
```

- Arrays are **ordered** (elements have indexes).
- Elements can be **any type**: strings, numbers, booleans, objects, even other arrays!
- Array **can contain mixed type elements**.

```ad-important
Negative indexes **don’t work with bracket access** (`arr[-1]` gives `undefined`), but **do work in some [[Array Methods|array methods]]**.
```

---

### ⚠️ Arrays Are Objects!

```js
typeof [1, 2, 3]; // "object"
```

So arrays **inherit** from `Array.prototype` and get many built-in methods.

```ad-important
This mean that it is a reference type.
```

---

## 📘 Basic Array Features

```js
let arr = [10, 20, 30];

console.log(arr[0]);      // 10 (first element)
console.log(arr.length);  // 3 (number of elements)
```

---

## 👉 Accessing item

Use `.at(index)` to access an item in an array — supports **negative indexing**:

```js
let arr = [10, 20, 30];
arr.at(0);    // 10
arr.at(-1);   // 30 (last item)
```

✅ Safer than `arr[index]` when working with negative numbers.

---

## 🛠 Common Array Methods

```ad-note
Arrays in JavaScript can work both as a queue and as a stack.
```

### ✅ Add / Remove Elements

|Method|What it does|
|---|---|
|`push(item)`|Add to **end**|
|`pop()`|Remove from **end**|
|`unshift(item)`|Add to **start**|
|`shift()`|Remove from **start**|

```js
let nums = [1, 2];
nums.push(3);    // [1, 2, 3]
nums.pop();      // [1, 2]
```

![[Pasted image 20250724160204.png|center|500]]

---

### 🔍 Search

```js
arr.includes(20);      // true
arr.indexOf(30);       // 2
```

---

### 🔁 Looping

```js
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

for (let value of arr) {
  console.log(value);  // cleaner
}

arr.forEach((value, index) => {
  console.log(index, value);
});
```

---

### 🧪 Transform

|Method|What it does|
|---|---|
|`map()`|Create new array by transforming each item|
|`filter()`|Keep only items that pass a test|
|`reduce()`|Collapse array to a single value|

```js
let nums = [1, 2, 3];
let squared = nums.map(x => x * x);    // [1, 4, 9]

let evens = nums.filter(x => x % 2 === 0); // [2]

let sum = nums.reduce((a, b) => a + b, 0); // 6
```

---

### 🔀 Other Useful Methods

|Method|What it does|
|---|---|
|`slice(start, end)`|Get part of the array|
|`splice(start, count, ...items)`|Remove or replace items|
|`join()`|Turn array into a string|
|`sort()`|Sort elements (can be tricky!)|
|`reverse()`|Reverse order|

```js
let a = ["a", "b", "c"];
a.slice(1);      // ["b", "c"]
a.splice(1, 1);  // ["a", "c"]
a.join("-");     // "a-b-c"
```

---

### 📏 length Property

- The `length` property automatically updates when we modify the array. To be precise, it is actually not the count of values in the array, but the greatest numeric index plus one.
- Another interesting thing about the `length` property is that it’s writable. If we increase it manually, nothing interesting happens. But if we decrease it, the array is truncated. The process is irreversible. So, the simplest way to clear the array is: `arr.length = 0;`.

---

## 🧵 Summary

|Feature|Description|
|---|---|
|Ordered|Yes|
|Can hold types|Any (mixed types allowed)|
|Zero-based index|Yes (`arr[0]` is the first item)|
|Mutable?|Yes (you can change values anytime)|