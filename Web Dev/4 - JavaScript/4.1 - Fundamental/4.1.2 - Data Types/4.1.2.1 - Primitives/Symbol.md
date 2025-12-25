---
tags:
  - js
  - datatype
  - fundamental
---

### 🔑 `Symbol` in JavaScript

`Symbol` is a **primitive data type** introduced in ES6. It’s used to create **unique identifiers** for object properties that **won’t clash** with others.

```ad-note
Most values in JavaScript support implicit conversion to a string. For instance, we can `alert` almost any value, and it will work. Symbols are special. They don’t auto-convert.
```

---

### ✅ Creating a Symbol

```js
const id = Symbol();
const id2 = Symbol();
console.log(id === id2); // false (each Symbol is unique)
```

Even if they have the same description:

```js
const sym1 = Symbol("desc");
const sym2 = Symbol("desc");
console.log(sym1 === sym2); // false
```

```ad-note
Symbol's description (also called a symbol name), mostly useful for debugging purposes.
```

---

### 💡 Why use Symbols?

1. **Avoid name clashes** — if many people add stuff to the same object, Symbols prevent stepping on each other's toes.
    
2. **Hide things** — Symbols are **not shown in for...in loops** or `Object.keys()`.
    
3. **Special behaviors** — JavaScript uses built-in Symbols for special things (like `Symbol.iterator` to make objects iterable).

---

### 📦 Use Case: Unique Object Keys

```js
const id = Symbol("id");

let user = {
  name: "Alice",
  [id]: 123  // symbol used as a key
};

console.log(user[id]); // 123
```

- The `id` property is **hidden from loops** and won't show up in `Object.keys()` or `for...in`.
    

```ad-note
You **must use bracket notation** to assign/read a symbol-keyed property.
```

---

### 🔍 Symbols Are Hidden

```js
for (let key in user) {
  console.log(key); // only "name" will be logged
}

console.log(Object.keys(user)); // ["name"]
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id)]
```

---

### ⚙️ Use in Libraries: Avoid Name Collisions

Symbols are often used in libraries or frameworks to:

- Store private/internal object metadata
    
- Avoid naming conflicts with user-defined properties
    

---

### 🧰 `Symbol.for()` – Shared Symbol Registry

```js
let sym1 = Symbol.for("shared");
let sym2 = Symbol.for("shared");

console.log(sym1 === sym2); // true
```

- `Symbol.for()` creates or retrieves a **global symbol**.
    
- Use this if you want the symbol to be shared across files/modules.
    
- This is like a **shared label** that different parts of the program can use to talk about the same concept **without hardcoding string keys**.

---

### 🧰 `Symbol.keyFor`

We have seen that for global symbols, `Symbol.for(key)` returns a symbol by name. To do the opposite – return a name by global symbol – we can use: `Symbol.keyFor(sym)`:

```js
// get symbol by name
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// get name by symbol
alert( Symbol.keyFor(sym) ); // name
alert( Symbol.keyFor(sym2) ); // id
```

The `Symbol.keyFor` internally uses the global symbol registry to look up the key for the symbol. So it doesn’t work for non-global symbols. If the symbol is not global, it won’t be able to find it and returns `undefined`.

---

### 🧠 Well-Known Symbols (Built-in behavior control)

JavaScript defines some built-in symbols to customize object behavior:

|Symbol|Used For|
|---|---|
|`Symbol.iterator`|Defines iteration (for `for...of`)|
|`Symbol.toPrimitive`|Custom object-to-primitive conversion|
|`Symbol.toStringTag`|Controls `Object.prototype.toString`|
|`Symbol.hasInstance`|Custom behavior for `instanceof`|

Example:

```js
const obj = {
  [Symbol.toPrimitive](hint) {
    return hint === "number" ? 42 : "hello";
  }
};

console.log(+obj);      // 42
console.log(`${obj}`);  // "hello"
```

---

### ❗️Symbols Are Not Strings

```js
let sym = Symbol("test");
console.log(typeof sym); // "symbol"
console.log(sym + ""); // ❌ TypeError: can't convert symbol to string
```

You must explicitly convert:

```js
console.log(sym.toString()); // "Symbol(test)"
```

---

### ✅ Summary

| Feature            | Symbol                        |
| ------------------ | ----------------------------- |
| Type               | Primitive                     |
| Use case           | Unique property keys          |
| Visibility         | Hidden from most enumerations |
| Equality           | Each symbol is unique         |
| `Symbol.for()`     | Creates/reuses global symbols | 
| Well-known symbols | Customize language behavior   |

----

There are three ways to access a Symbol created by a third-party library, depending on how the library author intended it to be used.

### 1. The Standard Way: The Library Exports the Symbol

If the library expects you to read or write to that specific Symbol property, they **must export the Symbol variable** so you can import it.

You cannot "recreate" the symbol; you must use the **exact same reference**.

**Library Code (`library.js`):**

```js
// The library creates the symbol
export const LIB_metadata = Symbol('metadata');

export const myObject = {
  [LIB_metadata]: "Secret Info",
  data: "Public Info"
};
```

**Your Code (`app.js`):**

```js
import { myObject, LIB_metadata } from './library.js';

// ❌ WRONG: Creating a new symbol won't work
const myGuess = Symbol('metadata');
console.log(myObject[myGuess]); // undefined

// ✅ CORRECT: Use the exported reference
console.log(myObject[LIB_metadata]); // "Secret Info"
```

---

### 2. The Global Registry: `Symbol.for()`

Sometimes, different libraries (or different parts of a huge app) need to agree on a specific Symbol without importing it from a single file. For this, JavaScript provides a **Global Symbol Registry**.

Instead of `Symbol()`, you use `Symbol.for('key')`.

- If a symbol with that key exists in the registry, it returns it.
    
- If not, it creates it globally.
    

**Library Code:**

```js
// Creates a global symbol keyed by 'app.user.id'
const userIdSym = Symbol.for('app.user.id');
const user = {
    [userIdSym]: 42
};
```

**Your Code:**

```js
// You don't need to import the variable!
// You just need to know the string key used in the registry.
const sameSym = Symbol.for('app.user.id');

console.log(user[sameSym]); // 42
console.log(sameSym === Symbol.for('app.user.id')); // true
```

> **Analogy:** `Symbol('foo')` is like a private key to a house (only the holder has it). `Symbol.for('foo')` is like a P.O. Box (anyone who knows the number can access it).

---

### 3. The "Hacker" Way: Reflection

If the library used a standard `Symbol()` (not global) and **did not export it**, they likely intended for that property to be private or internal.

However, if you _really_ need to access it (for debugging or patching), you can use `Object.getOwnPropertySymbols()`. This returns an array of all symbol keys on an object.

JavaScript

```js
import { hiddenObject } from './library.js';

// 1. Get all symbols on the object
const symbols = Object.getOwnPropertySymbols(hiddenObject);

// 2. Find the one you want (e.g., the first one)
const secretKey = symbols[0];

// 3. Access the data
console.log(hiddenObject[secretKey]); 
```

### Summary of Access Methods

| **Method**         | **Use Case**       | **Requirement**                                            |
| ------------------ | ------------------ | ---------------------------------------------------------- |
| **Export/Import**  | Standard API usage | You must import the variable from the library.             |
| **`Symbol.for()`** | Shared/Global keys | You must know the exact string key string.                 |
| **Reflection**     | Debugging/Hacking  | You retrieve the symbol from the object itself at runtime. | 
