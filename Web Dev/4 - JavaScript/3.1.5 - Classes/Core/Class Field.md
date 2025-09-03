---
tags: 
  - js
  - class
  - term
  - fundamental
---

## 🔹 What is a Class Field?

A **class field** is a property that belongs to each instance of a class, defined **directly inside the class body**, not in the constructor.

They make it easier to declare and initialize properties without boilerplate code.

---

## 🔹 Example: Public Class Fields

```js
class User {
  // class field
  name = "Anonymous";

  sayHi() {
    console.log(`Hi, I'm ${this.name}`);
  }

  // This is also a class field
   click = () => {
    alert(this.value);
  } 
}

let u = new User();
console.log(u.name); // "Anonymous"

u.name = "Alice";
u.sayHi(); // "Hi, I'm Alice"
```

⚡ Before class fields, you would have to write this in the constructor:

```js
constructor() {
  this.name = "Anonymous";
}
```

```ad-note
So, we just write “ = ” in the declaration, and that’s it.
```

---

## 🔹 Types of Class Fields

### 1. **Public Fields**

Declared normally, accessible everywhere.

```js
class Example {
  count = 0;
}
let e = new Example();
console.log(e.count); // 0
```

---

### 2. **Private Fields** (`#name`)

Start with `#`. They are **truly private** (not accessible outside the class).

```js
class User {
  #password = "secret"; // private field

  checkPassword(pw) {
    return this.#password === pw;
  }
}

let u = new User();
console.log(u.checkPassword("secret")); // true
console.log(u.#password); // ❌ SyntaxError
```

---

### 3. **Static Fields**

Belong to the class itself, not instances.

```js
class Counter {
  static count = 0;
  constructor() {
    Counter.count++;
  }
}

new Counter();
new Counter();
console.log(Counter.count); // 2
```

⚡ You can combine with `#` to create **private static fields**.

---

## 🔹 Difference Between Fields and Methods

- **Methods** go on the prototype (`User.prototype`).
    
- **Fields** are assigned to each instance directly.
    

```js
class Test {
  field = "I am a field";
  method() { return "I am a method"; }
}

let t = new Test();

console.log("field" in t); // true (own property)
console.log("method" in t); // true (but on prototype)

console.log(Object.hasOwn(t, "field"));   // true
console.log(Object.hasOwn(t, "method"));  // false
```

---

✅ **Summary**

- Class fields let you define properties outside the constructor.
    
- They can be **public**, **private (#)**, or **static**.
    
- Fields live **on instances**, while methods live **on the prototype**.
    