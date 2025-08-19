---
tags:
  - js
  - operator
  - fundamental
---

### ✨ Spread Syntax in JavaScript (`...`)

The **spread operator (`...`)** in JavaScript lets you **expand elements of an iterable** (like arrays, objects, or strings) into individual elements.

---

## 🔹 1. **Spread in Arrays**

### ✅ Copy an array

```js
const arr = [1, 2, 3];
const copy = [...arr];
```

### ✅ Merge arrays

```js
const a = [1, 2];
const b = [3, 4];
const merged = [...a, ...b]; // [1, 2, 3, 4]
```

### ✅ Add elements

```js
const extended = [0, ...arr, 4]; // [0, 1, 2, 3, 4]
```

---

## 🔹 2. **Spread in Objects**

### ✅ Clone an object

```js
const user = { name: "Alice" };
const clone = { ...user };
```

### ✅ Merge objects

```js
const defaults = { theme: "dark" };
const settings = { fontSize: 14 };
const config = { ...defaults, ...settings }; // { theme: "dark", fontSize: 14 }
```

> ⚠️ If keys overlap, the last one wins.

---

## 🔹 3. **Spread in Function Arguments**

```js
function sum(a, b, c) {
  return a + b + c;
}

const nums = [1, 2, 3];
console.log(sum(...nums)); // 6
```

---

## 🔹 4. **Spread Strings (into characters)**

```js
const chars = [..."hi"]; // ['h', 'i']
```

---

## ✅ Summary

|Use Case|Example|
|---|---|
|Clone array|`[...arr]`|
|Merge arrays|`[...a, ...b]`|
|Clone object|`{...obj}`|
|Merge objects|`{...a, ...b}`|
|Expand in args|`fn(...arr)`|
|Split string|`[..."text"]`|