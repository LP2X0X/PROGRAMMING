---
tags: 
 - js
 - overview
---

# 🔑 Big Picture

- In JavaScript, **functions**, **classes**, and **objects** are deeply connected.
    
- **Classes are just special functions** (syntactic sugar).
    
- **Objects are instances** created from functions/classes.
    
- Everything relies on the **prototype chain**.
    

---

# 1. Functions

- A **function** is both:
    
    - **callable** (you can execute it),
        
    - and an **object** (it has properties).
        
- Every function automatically gets a `.prototype` object when defined (except arrow functions).
    
- That `.prototype` is used as the blueprint for objects created with `new`.
    

Example:

```js
function Person(name) {
  this.name = name;
}
```

- `Person.prototype` → a plain object `{ constructor: Person }`
    
- `new Person("Alice")` → creates an object that delegates to `Person.prototype`
    

---

# 2. Objects

- An **object** is just a collection of key–value pairs with an internal hidden property `[[Prototype]]`.
    
- `[[Prototype]]` links to another object (or `null`).
    
- This link is used for **inheritance** (property/method lookup).
    

Example:

```js
const alice = new Person("Alice");

console.log(Object.getPrototypeOf(alice) === Person.prototype); // true
```

- If you access `alice.sayHello` and it’s not defined on `alice`, JavaScript looks up the prototype chain:  
    `alice → Person.prototype → Object.prototype → null`.
    

---

# 3. Classes

- A **class** is just syntax sugar for a constructor function + prototype methods.
    
- Under the hood:
    
    ```js
    class Person {
      constructor(name) {
        this.name = name;
      }
      sayHello() {
        console.log("Hello, I'm " + this.name);
      }
    }
    ```
    
    is equivalent to:
    
    ```js
    function Person(name) {
      this.name = name;
    }
    Person.prototype.sayHello = function() {
      console.log("Hello, I'm " + this.name);
    };
    ```
    
- Key facts:
    
    - `typeof Person === "function"`
        
    - `Person.prototype.constructor === Person`
        
    - `new Person("Alice")` creates an object with `[[Prototype]] = Person.prototype`
        

---

# 4. Relationships

Here’s the **chain of relations**:

1. **Class/Function definition**
    
    ```js
    class Person {}
    // is actually: function Person() {}
    ```
    
2. **Prototype link**
    
    - `Person.prototype` = object with methods.
        
    - `Person.prototype.constructor = Person`.
        
3. **Object creation**
    
    ```js
    const alice = new Person();
    ```
    
    - `alice.[[Prototype]] = Person.prototype`
        
4. **Prototype chain lookup**
    
    ```
    alice → Person.prototype → Object.prototype → null
    ```
    

---

# 5. Visual Diagram

```
           Function (callable + object)
                │
                ▼
      +-------------------+
      |   Person class    |   (actually a function)
      +-------------------+
          │        ▲
          │        │ constructor
          ▼
+------------------------+
|   Person.prototype     |  (object holding shared methods)
+------------------------+
          │
          ▼
+------------------------+
|   alice instance       |  (object created with `new`)
+------------------------+
```

---

# 6. Key Takeaways

- **Object** = instance with `[[Prototype]]`.
    
- **Function** = callable object, can act as a constructor.
    
- **Class** = cleaner syntax for constructor functions.
    
- `prototype` (on functions/classes) = blueprint for instances.
    
- `constructor` (inside class) = initializer code.
    
- `prototype.constructor` = reference back to the function/class.
    

---

👉 In other words:  
**Classes build on functions, functions create objects, and objects delegate behavior through prototypes.**

---

Would you like me to extend this into a **“mental model cheatsheet”** (quick reference with rules + diagrams) so you can use it whenever you’re stuck?