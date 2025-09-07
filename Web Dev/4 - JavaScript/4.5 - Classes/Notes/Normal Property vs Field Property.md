---
tags: 
 - js
 - distinguish
 - advance
---

## 🔹 1. Class property (traditional way)

Before class fields existed, you usually assigned properties **inside the constructor**:

```js
class User {
  constructor(name) {
    this.name = name; // normal property
  }
}
```

Here:

- `name` is just a **normal object property** added when the instance is created.
    
- It doesn’t live on the prototype — each instance gets its own `name` value.
    

---

## 🔹 2. Class field (modern syntax)

With class fields, you can declare the property directly inside the class body:

```js
class User {
  name = "guest"; // class field
}
```

This is basically **syntactic sugar** for writing it inside the constructor:

```js
class User {
  constructor() {
    this.name = "guest";
  }
}
```

So functionally, **a class field is also just a normal property on the instance**.  
The main differences are:

- **Cleaner syntax** (no need to clutter constructor).
    
- **Default values** can be declared inline.
    
- Can also be functions (arrow functions) that automatically bind `this`.
    

---

## 🔹 3. Methods (not properties!)

```js
class User {
  greet() { console.log("hi"); }
}
```

- `greet` is **not an instance property**.
    
- It lives on **User.prototype** and is shared across all instances.
    
- That’s why `this` binding issues happen when you pass methods around.
    

---

## ✅ So, difference between “normal property” and “field”:

- If you mean `this.name = ...` (constructor) vs `name = ...` (field):  
    ➝ No big difference, both create **instance properties**. Fields are just syntactic sugar with inline defaults.
    
- If you mean **field vs method**:  
    ➝ Big difference! Fields are **per-instance properties**, methods live on the **prototype**.
    