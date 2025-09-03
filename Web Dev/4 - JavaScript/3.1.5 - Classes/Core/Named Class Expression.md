---
tags: 
  - js
  - class
  - term
  - fundamental
---

A **named class expression** in JavaScript is a class expression where you give the class a name.

### Syntax

```js
let MyClass = class ClassName {
  constructor(value) {
    this.value = value;
  }
  show() {
    return this.value;
  }
};
```

Here:

- `MyClass` is the variable that holds the class.
    
- `ClassName` is the internal name of the class (only accessible **inside the class body**, not outside).
    

---

### Example

```js
let User = class UserHelper {
  constructor(name) {
    this.name = name;
  }

  printName() {
    // Can refer to UserHelper inside
    return UserHelper.format(this.name);
  }

  static format(str) {
    return str.toUpperCase();
  }
};

let u = new User("Alice");
console.log(u.printName()); // ALICE

console.log(typeof UserHelper); // ❌ ReferenceError (not visible outside)
```

---

### Why use a named class expression?

1. **Self-reference** inside the class (useful for recursion).
    
    ```js
    let LinkedList = class Node {
      constructor(value, next = null) {
        this.value = value;
        this.next = next instanceof Node ? next : null;
      }
    };
    ```
    
2. **Better stack traces** in debugging (the name shows up in error messages).
    

---

👉 So the key difference:

- **Anonymous class expression**: `let A = class {}`
    
- **Named class expression**: `let A = class B {}` (but `B` only exists inside the class body).
    