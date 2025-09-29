---
tags:
  - js
  - keyword
  - fundamental
---
- In JavaScript, `const` is a keyword used to declare **block-scoped constants**. Here's what you need to know:

---

### 🔑 **Key Characteristics of `const`**

1. **Block Scope** – Similar to `let`, a `const` is only accessible within the block it's declared in (e.g., inside `{}`).

2. **Must Be Initialized** – You **must assign a value** at the time of declaration:

   ```js
   const name = 'Alice'; // ✅
   const age;             // ❌ SyntaxError: Missing initializer
   ```

3. **Can't Be Reassigned** – You **cannot change the reference** once it's set:

   ```js
   const a = 5;
   a = 10; // ❌ TypeError
   ```

---

### 🧠 **But… `const` is not immutable!**

* If the value is an **object or array**, its **contents can still be modified**:

  ```js
  const user = { name: 'Alice' };
  user.name = 'Bob';         // ✅ allowed
  user = { name: 'Charlie' }; // ❌ TypeError
  ```

---

### ✅ **Good Practices**

* Use `const` **by default** unless you know the variable needs to change (then use `let`).
* Prefer `const` for:

  * Functions
  * Imports
  * Object and array declarations that won't be reassigned

---

### ✅ Example:

```js
const PI = 3.14;
const user = {
  name: "John"
};

user.name = "Jane"; // ✅ This is allowed
```
