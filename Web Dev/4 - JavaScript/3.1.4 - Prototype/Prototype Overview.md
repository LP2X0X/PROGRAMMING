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
    

</br>

----

</br>

# Other overview

## 1. Functions and `prototype`

- Every function in JS (except arrow functions) automatically gets a special property called `.prototype` when created.
    
- This `.prototype` is an object that is **used as the blueprint for instances** when the function is called with `new`.
    
- Example:
    
    ```js
    function Person(name) {
      this.name = name; // instance property
    }
    
    Person.prototype.sayHi = function () {
      console.log("Hi, I'm " + this.name);
    };
    
    let p = new Person("Alice");
    p.sayHi(); // "Hi, I'm Alice"
    ```
    
    - `p.__proto__ === Person.prototype` ✅
        
    - The instance (`p`) does **not** have `.prototype`; instead, it has `__proto__` linking to the function’s `.prototype`.
        

---

## 2. `__proto__` (a.k.a. `[[Prototype]]`)

- Every object in JS has an internal hidden slot called `[[Prototype]]`.
    
- The standard way to access it is `Object.getPrototypeOf(obj)` (not `__proto__` directly, though `__proto__` works in most engines).
    
- `[[Prototype]]` is what makes **prototype chain lookup** possible.
    
    - When you access a property on an object, JS will:
        
        1. Look at the object itself.
            
        2. If not found, follow its `[[Prototype]]` (`__proto__`) link.
            
        3. Keep going until `null` (usually at `Object.prototype`).
            

Example:

```js
let obj = {};
console.log(obj.__proto__ === Object.prototype); // true
```

---

## 3. Classes and Prototype

- A `class` in JS is just **syntactic sugar** over constructor functions.
    
- Example:
    
    ```js
    class Animal {
      speak() { console.log("generic sound"); }
    }
    
    let a = new Animal();
    
    console.log(a.__proto__ === Animal.prototype); // true
    console.log(Animal.prototype.constructor === Animal); // true
    ```
    
- Methods you define inside a class body (`speak` above) are automatically placed on `ClassName.prototype`.
    
- Static methods are put directly on the constructor function itself, not the prototype:
    
    ```js
    class Animal {
      static info() { console.log("static"); }
    }
    
    Animal.info(); // works
    let a = new Animal();
    // a.info(); // ❌ doesn’t work
    ```
    

---

## 4. The relationship map

Here’s the chain for a typical case:

```
   class Foo {}
       ↓
 Foo.prototype (where instance methods live)
       ↓
 Foo.prototype.__proto__ === Object.prototype
       ↓
 Object.prototype.__proto__ === null
```

And for an instance:

```
 let obj = new Foo();

 obj.__proto__ === Foo.prototype
 Foo.prototype.constructor === Foo
```

For the constructor function (or class itself):

```
 Foo.__proto__ === Function.prototype
 Function.prototype.__proto__ === Object.prototype
```

---

## 5. Quick analogy

- `.prototype` → belongs to **functions/classes**, used as a **template** for new objects.
    
- `__proto__` → belongs to **objects/instances**, points to the object’s **parent prototype**.
    
- Together, they form the **prototype chain**.
    


</br>

----

</br>

# My version


### 🔑 Prototypes in JavaScript (Overview)

- **Every object** in JavaScript has an internal reference called `__proto__`, which points to another object (its prototype).
    
- **Functions** are special objects: in addition to `__proto__`, they also have a `.prototype` property.
    
- Both `__proto__` and `.prototype` hold **objects** and are the foundation of JavaScript’s **prototypal inheritance system**.
    

---

### 📌 How they work

1. **Prototype vs `__proto__`**
    
    - `.prototype`: exists only on functions, used as the template for new objects created with `new`.
        
    - `__proto__`: exists on all objects, pointing to the prototype it was created from.
        
2. **Linking**
    
    - When you set `Function.prototype`, any instance created with `new Function()` will have its `__proto__` linked to that prototype.
        
    - Changing the function’s `.prototype` also changes the `__proto__` of **new instances**, but not existing ones.
        
3. **Constructor property**
    
    - By default, `Function.prototype` includes a `constructor` property that points back to the function itself.
        
    - This can be overridden (intentionally or by accident).
        
4. **Instance vs Prototype properties**
    
    - Properties defined directly in the constructor (`this.name = ...`) are **instance properties** (unique per object).
        
    - Methods defined in the class body or added manually to `Function.prototype` are **shared prototype methods** (one copy, shared by all instances).
        

---

### ⚡ Important Notes You Missed

- **Lookup chain (prototype chain):**  
    When you access a property on an object, JS looks for it:
    
    1. In the object itself.
        
    2. If not found, in its `__proto__`.
        
    3. Keeps going up until it reaches `Object.prototype`, whose `__proto__` is `null`.
        
- **You can add both methods _and_ properties to prototypes:**  
    Although typically only methods are put on the prototype, nothing stops you from putting shared data too (but this can lead to unexpected mutations if objects share it).
    
- **`__proto__` vs `[[Prototype]]`:**  
    `__proto__` is just an accessor for the internal `[[Prototype]]` slot. Modern code should prefer `Object.getPrototypeOf()` and `Object.setPrototypeOf()` instead of directly using `__proto__`.
    
- **Performance consideration:**  
    Instance properties are faster to access, but duplicate across objects. Prototype methods save memory since they’re shared.
    
- **Inheritance:**  
    Prototype chaining is what makes `class` and `extends` work. Under the hood, it just sets the `__proto__` of one prototype object to another.
    

---

👉 So a sharper version of your summary could be:

> Every object in JavaScript has a `[[Prototype]]` (accessible via `__proto__`), which forms the basis of prototypal inheritance. Functions also have a `.prototype` property, which is used as the prototype for objects created with `new`. Instance properties are stored directly on each object, while shared methods (and sometimes properties) are stored on the prototype. Property lookups follow the prototype chain until `Object.prototype`. This system enables inheritance, method sharing, and memory efficiency.

---
