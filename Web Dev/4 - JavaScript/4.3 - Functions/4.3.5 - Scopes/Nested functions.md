---
tags:
  - js
  - scope
  - fundamental
---

### 🔹 What is a Nested Function?

A **nested function** is a function **defined inside another function**.

### ✅ Example:

```js
function sayHiBye(firstName, lastName) {
  function getFullName() {
    return firstName + " " + lastName;
  }

  alert("Hello, " + getFullName());
  alert("Bye, " + getFullName());
}
```

- `getFullName` is a **helper function**.
- It **accesses outer variables** (`firstName`, `lastName`) from `sayHiBye`.

---

## 🔐 Closures: Returning Nested Functions

### ✅ Example: `makeCounter`

```js
function makeCounter() {
  let count = 0;

  return function() {
    return count++;
  };
}

let counter = makeCounter();

alert(counter()); // 0
alert(counter()); // 1
alert(counter()); // 2
```

### 🔍 How it works:

- The inner function **remembers** the outer variable `count` — even after `makeCounter` finishes.
- This is called a **closure**.
- `count` is private to the returned function — it persists between calls.

---

## 🔄 Are Multiple Counters Independent?

```js
let counter1 = makeCounter();
let counter2 = makeCounter();

counter1(); // 0
counter1(); // 1

counter2(); // 0
```

Yes — each call to `makeCounter()` creates a **new `count` variable in a new scope**.

---

## 💡 Practical Uses

- **Encapsulation**: Hides internal state (like `count`).
- **Stateful functions**: Useful for counters, random generators, caching, etc.
- **Test helpers**: Generate consistent or isolated behavior.

---

## 🧠 Summary

|Concept|Description|
|---|---|
|Nested function|A function inside another function|
|Closure|Inner function keeps access to outer variables|
|makeCounter|Returns a function that “remembers” `count`|
|Independence|Each returned function has its own state|