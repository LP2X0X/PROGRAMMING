---
tags:
  - js
  - term
---

### ✅ What is `thisArg`?

In JavaScript, `thisArg` is a term used in **many higher-order functions** (like `Array.prototype.map`, `forEach`, `filter`, `Array.from`, etc.) to mean:

> "**The value to use as `this` inside the callback function.**"

---

### ✅ Why is it needed?

In JavaScript, the value of `this` **depends on how** a function is **called**, not where it is defined.

Sometimes, you want to control what `this` refers to inside a callback. That’s where `thisArg` comes in.

---

### ✅ Where do you use `thisArg`?

Functions that accept a callback often allow a second argument — `thisArg` — to **explicitly set** the value of `this` inside the callback.

Examples:

```js
arr.map(callback, thisArg)
arr.forEach(callback, thisArg)
Array.from(arrayLike, mapFn, thisArg)
```

---

### ✅ General Example

```js
const context = { prefix: "Item:" };

[1, 2, 3].map(function (val) {
  return this.prefix + val;
}, context);

// Output: ["Item:1", "Item:2", "Item:3"]
```

Here, `this` inside the callback refers to the `context` object — because we passed it as `thisArg`.

---

### ❌ Arrow Functions Ignore `thisArg`

Arrow functions don’t have their own `this` — they inherit it from the surrounding scope, so `thisArg` has **no effect**:

```js
[1, 2, 3].map((val) => this.prefix + val, context); // `this.prefix` is undefined
```

Use regular `function () {}` syntax if you need `thisArg`.

---

### ✅ Summary

| Term      | Meaning                                                   |
| --------- | --------------------------------------------------------- |
| `this`    | Refers to the context object in a function                |
| `thisArg` | Value you pass to set `this` in a callback                |
| Works in  | `map`, `forEach`, `filter`, `Array.from`, etc.            |
| Use case  | Needed when the callback uses `this` and you want control |

---

### Reference
https://javascript.info/array-methods#most-methods-support-thisarg