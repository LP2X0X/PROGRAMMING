---
tags: 
 - js
 - property
---

## 1. No real `protected` in JS

Unlike languages like Java, C#, or C++, **JavaScript has only two true access levels** in classes:

- **Public** (default) → accessible everywhere.
    
- **Private** (with `#field`) → accessible only inside the declaring class, **not even in subclasses**.
    

So **protected** (accessible inside the class + subclasses, but not outside) does **not exist natively** in JS.

---

## 2. How Developers Simulate "Protected"

Since ES doesn’t have `protected`, people use **naming conventions** or **workarounds**:

### (a) By convention

```js
class Parent {
  constructor() {
    this._protectedValue = 42; // convention: _prefix means "don’t touch from outside"
  }
}

class Child extends Parent {
  show() {
    console.log(this._protectedValue); // ✅ allowed in subclass
  }
}

const c = new Child();
c.show();          // 42
console.log(c._protectedValue); // 😬 still accessible from outside
```

Here `_protectedValue` is just a public property, but developers agree to treat it as “protected”.

---

### (b) Using closures (simulated protected)

You can use `WeakMap` or factory functions to hide state, but then subclasses **can’t directly access** it—so it behaves more like `private`.

```js
const _secret = new WeakMap();

class Parent {
  constructor() {
    _secret.set(this, 100);
  }
  _getSecret() { return _secret.get(this); }
}

class Child extends Parent {
  reveal() {
    return this._getSecret(); // subclass can still access through a method
  }
}
```

This works, but now it’s more like "private + protected accessor".

---

## 3. Why No Native Protected?

TC39 (the ECMAScript committee) intentionally skipped `protected` because:

- It complicates the language.
    
- JS philosophy leans on **flexibility and conventions** rather than rigid access controls.
    
- With `#private`, you can always make something private and expose protected-like accessors manually.
    

---

✅ **Summary**:

- JavaScript has **public** and **private**, but no real **protected**.
    
- Developers often simulate `protected` by **using `_` prefix** or **providing protected-style accessors**.
    
- Subclasses can access `_protected` by convention, but technically anyone can.
    