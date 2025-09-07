---
tags: 
 - js
 - distinguish
 - advance
---

```js
class User {
  constructor(name) {
    this.name = name;   // property
    this.age = 18;      // property
  }

  greet() {            // method (on prototype)
    console.log("Hi " + this.name);
  }
}

let u = new User("Alice");

console.log(u.name); // "Alice"
console.log(u.age);  // 18
```

Here:

- `this.name` and `this.age` → **properties** (live directly on the instance `u`).
    
- `greet` → **method** (lives on `User.prototype`, shared across instances).
    

---

### 🔍 Checking with `Object.hasOwn`

```js
console.log(Object.hasOwn(u, "name"));   // true
console.log(Object.hasOwn(u, "greet"));  // false (because greet is on prototype)
```

---

### ⚡ Difference from Class Fields

Declaring in constructor vs. declaring as a class field:

```js
class A {
  name; // class field
  constructor(age) {
    this.age = age; // property via constructor
  }
}

let obj = new A(20);
console.log(obj); // { name: undefined, age: 20 }
```

- `name` → class field (also becomes a property on the instance, initialized at construction).
    
- `age` → property assigned in constructor.
    

Both end up as **instance properties**, just declared in different ways.

---

👉 So the short answer:  
**Yes, anything you assign to `this` in the constructor is a property of the instance.**