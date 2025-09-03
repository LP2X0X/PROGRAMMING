---
tags: 
 - js
 - prototype
 - overview
 - advance
---

# 🏗️ Prototypes in JavaScript — Complete Overview

---

## 1. 🔹 Every Object Has a Prototype (`__proto__`)

- In JS, **all objects (except one)** have an internal hidden property called `[[Prototype]]` (commonly accessible via `__proto__`).
    
- This points to another object — its _prototype_.
    
- If you try to read a property and it’s not on the object itself, JS looks up the prototype chain (`__proto__`, then `__proto__.__proto__`, etc.).
    

```js
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true
```

---

## 2. 🔹 The Root: `Object.prototype`

- `Object.prototype` is the top of most prototype chains.
    
- It provides basic methods like:
    
    - `.toString()`
        
    - `.hasOwnProperty()`
        
    - `.valueOf()`
        
- `Object.prototype.__proto__` is `null` → end of the chain.
    

```js
console.log(Object.prototype.__proto__); // null
```

---

## 3. 🔹 Functions Are Special

- Functions are **objects** too → so they also have `__proto__`.
    
- But functions also have an extra property: **`.prototype`**.
    

### Difference:

- **`.prototype`** → exists only on functions, used when creating new objects with `new`.
    
- **`__proto__`** → exists on every object (including functions themselves).
    

```js
function Foo() {}
console.log(typeof Foo.prototype); // "object"
console.log(Foo.__proto__ === Function.prototype); // true
```

---

## 4. 🔹 Function’s `prototype` Property

- When you define a function, JS automatically creates a `.prototype` object:
    
    ```js
    function Person(name) { this.name = name; }
    console.log(Person.prototype); 
    // { constructor: Person }
    ```
    
- When you use `new Person()`, the new object’s `__proto__` is set to `Person.prototype`.
    
- Methods added to `Person.prototype` are shared across all instances.
    

```js
Person.prototype.sayHi = function() { console.log("Hi!"); };
const p = new Person("Alice");
p.sayHi(); // works because p.__proto__ === Person.prototype
```

---

## 5. 🔹 Special Built-In Prototypes

All built-in constructors come with their own `.prototype`:

- `Object.prototype` → root for normal objects
    
- `Array.prototype` → has array methods (`push`, `map`, etc.)
    
- `Function.prototype` → has function methods (`call`, `apply`, `bind`)
    
- `Date.prototype`, `RegExp.prototype`, etc.
    

Example:

```js
const arr = [1, 2, 3];
console.log(arr.__proto__ === Array.prototype); // true
console.log(Array.prototype.__proto__ === Object.prototype); // true
```

---

## 6. 🔹 The Prototype Chain Example

```js
function Foo() {}
const foo = new Foo();

console.log(foo.__proto__ === Foo.prototype);          // true
console.log(Foo.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__);               // null
```

---

## 7. 🛠️ Best Practices

- ✅ Prefer **class syntax** for readability — it’s just syntactic sugar for working with `.prototype`.
    
- ✅ Use `.hasOwnProperty()` to check if a property exists directly on the object (not from prototype).
    
- ✅ Don’t overwrite `Object.prototype` — it can break the entire runtime.
    
- ✅ Add methods to `.prototype` (or `class`) instead of recreating them inside constructors.
    

---

## 🏆 TL;DR

- **All objects** have a hidden `__proto__`.
    
- **Functions** also have `.prototype`, used for creating instances with `new`.
    
- **`Object.prototype`** is the root of most chains (`__proto__ === null`).
    
- Built-in types (`Array`, `Function`, `Date`, etc.) have their own `.prototype`.
    
- Together, these form the **prototype chain** → the backbone of inheritance in JavaScript.
    