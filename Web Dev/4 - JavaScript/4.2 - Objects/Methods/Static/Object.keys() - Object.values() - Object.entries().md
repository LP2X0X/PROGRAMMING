---
tags:
  - js
  - object
  - method
  - static
---

## 🧩 1. `Object.keys(obj)`

### 🔹 Returns:

An array of the **object’s own enumerable property names** (keys).

### ✅ Example:

```js
const person = { name: "Alice", age: 30 };
console.log(Object.keys(person));  // ["name", "age"]
```

---

## 🧩 2. `Object.values(obj)`

### 🔹 Returns:

An array of the **object’s own enumerable property values**.

### ✅ Example:

```js
const person = { name: "Alice", age: 30 };
console.log(Object.values(person));  // ["Alice", 30]
```

---

## 🧩 3. `Object.entries(obj)`

### 🔹 Returns:

An array of `[key, value]` pairs.

### ✅ Example:

```js
const person = { name: "Alice", age: 30 };
console.log(Object.entries(person));
// [ ["name", "Alice"], ["age", 30] ]

const arr = ["a", "b", "c"];

for (let [index, item] of arr.entries()) {
  console.log(index, item);
}
```

---

## 🔄 4. Looping with `entries`

```js
const obj = { a: 1, b: 2, c: 3 };

for (const [key, value] of Object.entries(obj)) {
  console.log(`${key} → ${value}`);
}
// a → 1
// b → 2
// c → 3
```

---

## 🔄 5. Convert back to object from `entries`

```js
const entries = [["name", "Alice"], ["age", 30]];
const obj = Object.fromEntries(entries);
console.log(obj); // { name: "Alice", age: 30 }
```

---

## 🔍 Comparison Table

|Method|Returns|Use Case|
|---|---|---|
|`Object.keys()`|Array of keys|Looping over property names|
|`Object.values()`|Array of values|Getting all property values|
|`Object.entries()`|Array of `[key, value]` pairs|Looping over full object content|
|`Object.fromEntries()`|Object from entries|Convert back from entries format|

---

## 💡 Notes

- These methods only work on **own enumerable** properties (not inherited ones).
    
- The order of keys is the same as in a `for...in` loop (except `for...in` also gets inherited properties).
    