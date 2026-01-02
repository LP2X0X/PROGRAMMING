---
tags: 
 - js
 - object
 - method
 - static
---

## 🧊 `Object.freeze()` — Overview

`Object.freeze()` makes an object **immutable at runtime**.

Once an object is frozen:

- ❌ You cannot add new properties
    
- ❌ You cannot delete properties
    
- ❌ You cannot change existing property values
    
- ❌ You cannot reconfigure property descriptors
    

```js
const obj = Object.freeze({ a: 1 });
```

---

## 🔒 What Exactly Gets Frozen

```js
const user = Object.freeze({
  name: "Long",
});

user.name = "Pham"; // ❌ blocked
user.age = 20;      // ❌ blocked
delete user.name;   // ❌ blocked
```

The object becomes:

- **Non-extensible**
    
- All properties become **non-writable**
    
- All properties become **non-configurable**
    

---

## ⚠️ Strict Mode vs Non-Strict Mode

### Non-Strict Mode

```js
const o = Object.freeze({ x: 1 });
o.x = 2; // ❌ fails silently
```

### Strict Mode

```js
"use strict";
const o = Object.freeze({ x: 1 });
o.x = 2; // ❌ TypeError
```

Strict mode is strongly recommended when using `Object.freeze()`.

---

## 🧠 Shallow Freeze (Very Important)

`Object.freeze()` is **shallow**.

```js
const state = Object.freeze({
  config: { theme: "dark" },
});

state.config.theme = "light"; // ✅ allowed
```

Only the **top-level object** is frozen.  
Nested objects remain mutable unless frozen separately.

---

## 🔍 Checking Frozen State

```js
Object.isFrozen(obj); // true or false
```

Useful for debugging and assertions.

---

## 🧪 Relationship to Other Object Locks

|Method|Can Add|Can Delete|Can Modify|
|---|---|---|---|
|`Object.preventExtensions()`|❌|✅|✅|
|`Object.seal()`|❌|❌|✅|
|`Object.freeze()`|❌|❌|❌|

`Object.freeze()` is the **strongest** form of object locking.

---

## 🚫 What `Object.freeze()` Does NOT Do

- ❌ Does not deep-freeze objects
    
- ❌ Does not clone objects
    
- ❌ Does not improve performance
    
- ❌ Does not make values immutable (only object structure)
    

```js
const frozen = Object.freeze({ arr: [1, 2] });
frozen.arr.push(3); // ✅ allowed
```

---

## ✅ Typical Use Cases

✔ Configuration objects  
✔ Constants shared across modules  
✔ Defensive programming (catch mutations early)  
✔ Public API return values

🚫 Not suitable for frequently updated data

---

## 🧾 Summary

- `Object.freeze()` enforces **runtime immutability**
    
- It prevents **writes, deletes, and extensions**
    
- It is **shallow**
    
- Errors are only thrown in **strict mode**
    
- Best used for **stable, shared objects**
    
