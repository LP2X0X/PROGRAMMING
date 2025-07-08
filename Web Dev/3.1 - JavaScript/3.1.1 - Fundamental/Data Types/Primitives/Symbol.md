---
tags: js, datatype, fundamental
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

|Feature|Symbol|
|---|---|
|Type|Primitive|
|Use case|Unique property keys|
|Visibility|Hidden from most enumerations|
|Equality|Each symbol is unique|
|`Symbol.for()`|Creates/reuses global symbols|
|Well-known symbols|Customize language behavior|
