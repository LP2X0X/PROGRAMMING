---
tags: 
  - js
  - function
  - fundamental
---

Before ES6 `class`, the main way to create multiple objects of the same “type” was with a **constructor function**.

### 🔹 Defining a Constructor Function

```js
function Person(name, age) {
  this.name = name;   // instance property
  this.age = age;
  this.greet = function() {  // instance method (not shared)
    console.log(`Hi, I'm ${this.name}`);
  };
}

const p1 = new Person("Alice", 25);
const p2 = new Person("Bob", 30);

console.log(p1.name); // "Alice"
console.log(p2.name); // "Bob"
```

- By convention, constructor function names are **PascalCase** (e.g., `Person`, `Car`, `Book`).
    
- When you call it with `new`:
    
    1. A new empty object is created.
        
    2. The object is linked to the constructor’s `.prototype`.
        
    3. The function body runs, binding `this` to the new object.
        
    4. The object is returned automatically (unless you explicitly return another object).
        

---

### 🔹 Adding Methods via Prototype

To avoid duplicating methods per instance, attach them to the constructor’s prototype:

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  console.log(`Hi, I'm ${this.name}`);
};

const p1 = new Person("Alice", 25);
const p2 = new Person("Bob", 30);

console.log(p1.greet === p2.greet); // true (shared method)
```

---

### 🔹 Equivalent Class Syntax

With ES6, `class` is just syntactic sugar over constructor functions:

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {   // goes to Person.prototype
    console.log(`Hi, I'm ${this.name}`);
  }
}
```

Internally, it works the same way — a `constructor` function is created and methods go to `.prototype`.

---

### 🔹 Special Notes

- If you call a constructor function **without `new`**, `this` will be `undefined` in strict mode or the global object in sloppy mode (not what you want).
    
    ```js
    const p = Person("Alice", 25); // ❌ no 'new' → modifies globalThis
    ```
    
- You can enforce `new` with:
    
    ```js
    function Person(name) {
      if (!(this instanceof Person)) {
        return new Person(name);
      }
      this.name = name;
    }
    ```
    

---

## ✅ Summary

- Constructor function = normal function used with `new` to create objects.
    
- Sets up `this` = new object.
    
- Links new object to `FunctionName.prototype`.
    
- Still widely used under the hood (even `class` uses them).
    