---
tags:
  - js
  - function
  - method
---

## 🔧 What Are `call()` and `apply()`?

Both `call()` and `apply()` are **methods of all JavaScript functions**. They allow you to **manually set the `this` value** when calling a function.

They differ in how they **pass arguments** to the function.

---

## 📌 Syntax

### `call(thisArg, arg1, arg2, ...)`

Calls a function with `this` set to `thisArg`, followed by individual arguments.

```js
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: "Alice" };
greet.call(person, "Hello", "!"); // Hello, Alice!
```

Another example:

```js
function sayHi() {
  alert(this.name);
}

let user = { name: "John" };
let admin = { name: "Admin" };

// use call to pass different objects as "this"
sayHi.call( user ); // John
sayHi.call( admin ); // Admin
```

---

### `apply(thisArg, [argsArray])`

Same as `call()`, but arguments are passed as a **single array**.

```js
greet.apply(person, ["Hi", "."]); // Hi, Alice.
```

---

## ✅ Best Use Cases

| Use Case                                               | Use       |
| ------------------------------------------------------ | --------- |
| Known number of arguments                              | `call()`  |
| Arguments are already in an array or array-like object | `apply()` |

---

## 🆕 With ES6 and Spread Operator

You can replace `apply()` with the spread syntax for clarity:

```js
greet.call(person, ...["Hi", "!"]); // same as apply
```

This makes `apply()` less common in modern code.

---

## 🔄 Use in Method Borrowing

You can borrow methods from one object and use them on another:

```js
const arrayLike = {
  0: "a",
  1: "b",
  length: 2
};

const realArray = Array.prototype.slice.call(arrayLike);
console.log(realArray); // ["a", "b"]
```

---

## 🧠 Use in Decorators

Manually forwarding calls while preserving `this`:

```js
function wrapper(func) {
  return function(...args) {
    console.log("Before call");
    return func.call(this, ...args); // preserve original context
  }
}
```

---

## ⚠️ Important Notes

1. **Always be mindful of the value of `this`.** Using `call()` and `apply()` lets you control it precisely.
    
2. **Arrow functions do not have their own `this`**, so you can't use `call()` or `apply()` to change their context.
    

---

## 🔐 Summary Table

| Feature        | `call()`                     | `apply()`       |
| -------------- | ---------------------------- | --------------- |
| Use            | Manual function invocation   | Same            |
| `this` control | Yes                          | Yes             |
| Arguments      | List (comma-separated)       | Array           |
| Modern alt     | `Function(...args)` + spread | Yes (`...args`) |