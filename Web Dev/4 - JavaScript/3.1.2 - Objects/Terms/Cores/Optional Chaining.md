---
tags:
  - js
  - syntax
  - fundamental
---

### 🔍 Optional Chaining (`?.`) in JavaScript

**Optional chaining (`?.`)** allows you to **safely access deeply nested properties** without having to check each level for `null` or `undefined`.

---

### ✅ Basic Syntax

```js
obj?.prop
obj?.[expr]
func?.(args)
```

Instead of throwing an error when accessing properties of `undefined` or `null`, it returns `undefined`.

---

###  ⚠️ Don’t overuse the optional chaining

- We should use `?.` only where it’s ok that something doesn’t exist.

- For example, if according to our code logic `user` object must exist, but `address` is optional, then we should write `user.address?.street`, but not `user?.address?.street`.

- Then, if `user` happens to be undefined, we’ll see a programming error about it and fix it. Otherwise, if we overuse `?.`, coding errors can be silenced where not appropriate, and become more difficult to debug.

---

### 📦 Example 1: Property Access

```js
let user = {};
console.log(user.address?.city); // undefined (no error)
```

Without optional chaining:

```js
console.log(user.address.city); // ❌ TypeError: Cannot read property 'city' of undefined
```

---

### 📦 Example 2: Array Access

- The `?.[]` syntax also works, if we’d like to use brackets `[]` to access properties instead of dot `.`.

```js
let users = null;
console.log(users?.[0]); // undefined
```

---

### 📦 Example 3: Method Calls

- Then `?.()` checks the left part: if the function exists, then it runs. Otherwise the evaluation stops without errors.

```js
let user = {
  greet() {
    return "Hello!";
  }
};

console.log(user.greet?.()); // "Hello!"
console.log(user.farewell?.()); // undefined (no error)
```

---

### 📦 Example 4: Dynamic Keys

```js
let key = "name";
let obj = { name: "Alice" };

console.log(obj?.[key]); // "Alice"
```

---

### ⚠️ Cannot Use on Left Side of Assignment

```js
let obj = {};
obj?.name = "Bob"; // ❌ SyntaxError
```

Optional chaining is **only for reading/accessing**, not for writing.

---

### 🔄 Combine with Nullish Coalescing (`??`)

```js
let user = {};
let name = user.info?.name ?? "Anonymous";
console.log(name); // "Anonymous"
```

---

### 🧠 Summary

|Use case|With optional chaining|Without|
|---|---|---|
|Deep property|`a?.b?.c`|`a && a.b && a.b.c`|
|Array element|`arr?.[0]`|`arr && arr[0]`|
|Function call|`obj.func?.()`|`obj.func && obj.func()`|
