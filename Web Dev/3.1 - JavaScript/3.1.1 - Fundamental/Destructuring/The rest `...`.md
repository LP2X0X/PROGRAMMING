---
tags: js, destructuring, term, fundamental
---

In JavaScript, **"the rest"** typically refers to the **rest syntax** (`...`) used in functions, arrays, or objects to collect remaining values.

---

## ✅ 1. **Rest Parameters (Functions)**

Used to gather **remaining arguments** into an array:

```js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3); // 6
```

Here, `...numbers` is the rest parameter.

---

## ✅ 2. **Rest with Arrays (Destructuring)**

```js
const [first, ...rest] = [1, 2, 3, 4];
console.log(first); // 1
console.log(rest);  // [2, 3, 4]
```

---

## ✅ 3. **Rest with Objects (Destructuring)**

```js
const { name, ...info } = { name: "Alice", age: 25, city: "Seoul" };
console.log(name); // "Alice"
console.log(info); // { age: 25, city: "Seoul" }
```