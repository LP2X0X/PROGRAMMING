---
tags: 
 - js
 - object
 - property
 - fundamental
---

In JavaScript, **getters** and **setters** are special methods that let you control access to an object’s properties. They’re defined inside objects or classes and let you run logic when a property is read (`get`) or written (`set`).

---

## 🔹 Syntax

### Object literal

```js
let user = {
  firstName: "Long",
  lastName: "Pham",

  // Getter
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },

  // Setter
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(" ");
  }
};

console.log(user.fullName); // "Long Pham" (calls getter)
user.fullName = "Nguyen An"; // (calls setter)
console.log(user.firstName); // "Nguyen"
```

---

### Class

```js
class User {
  constructor(name) {
    this._name = name; // use "_" to indicate private-like
  }

  // Getter
  get name() {
    return this._name.toUpperCase();
  }

  // Setter
  set name(value) {
    if (value.length < 3) {
      console.log("Name too short!");
      return;
    }
    this._name = value;
  }
}

let u = new User("Long");
console.log(u.name);  // "LONG" (getter)
u.name = "An";        // "Name too short!" (setter logic)
u.name = "Khang";
console.log(u.name);  // "KHANG"
```

---

## 🔹 Behind the scenes

- **Getter** → behaves like a property but actually calls a function when you read it.
    
- **Setter** → behaves like a property but actually calls a function when you assign to it.
    
- Together, they make a **computed property** that looks like a normal field.
    

---

## 🔹 Defining with `Object.defineProperty`

You can also define them explicitly:

```js
let obj = {};
Object.defineProperty(obj, "x", {
  get() { return this._x || 0; },
  set(value) { this._x = value; }
});

obj.x = 42;
console.log(obj.x); // 42
```

---

✅ **Best use cases**:

- Encapsulation (control access to internal data).
    
- Validation before setting a value.
    
- Creating computed properties.
    
- Backward compatibility (change internal logic without changing the API).
    