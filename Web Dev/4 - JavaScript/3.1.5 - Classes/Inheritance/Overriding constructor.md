---
tags: 
 - js
 - class
 - inheritance
 - override
---

## 🔹 Default behavior

If a class doesn’t define its own `constructor`, JavaScript creates one automatically:

- **Base class (no `extends`)** → constructor just initializes `this`.
    
- **Derived class (`extends`)** → constructor automatically calls `super(...args)`.
    

Example:

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {} // no constructor here

const d = new Dog("Rex");
console.log(d.name); // Rex
```

Here, `Dog` has no `constructor`, but JS creates one like this internally:

```js
constructor(...args) {
  super(...args);
}
```

---

## 🔹 Overriding constructor

You can override the constructor by explicitly writing your own.  
But in a derived class, you **must call `super(...)` before using `this`**.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);        // call parent constructor
    this.breed = breed; // now safe to use "this"
  }
}

const d = new Dog("Rex", "Labrador");
console.log(d.name);  // Rex
console.log(d.breed); // Labrador
```

```ad-note
In JavaScript, there’s a distinction between a constructor function of an inheriting class (so-called “derived constructor”) and other functions. A derived constructor has a special internal property `[[ConstructorKind]]:"derived"`. That’s a special internal label.

That label affects its behavior with `new`.

- When a regular function is executed with `new`, it creates an empty object and assigns it to `this`.
- But when a derived constructor runs, it doesn’t do this. It expects the parent constructor to do this job.

So a derived constructor must call `super` in order to execute its parent (base) constructor, otherwise the object for `this` won’t be created. And we’ll get an error.
```

---

## 🔹 What happens if you skip `super`?

```js
class Dog extends Animal {
  constructor(name, breed) {
    this.breed = breed; // ❌ ReferenceError: Must call super constructor
  }
}
```

Because in a derived class, `this` doesn’t exist until the parent’s constructor runs.

---

## 🔹 Overriding in base classes

If the class does **not extend** another class, you don’t need `super` at all.

```js
class Car {
  constructor(model) {
    this.model = model;
  }
}

const c = new Car("Tesla");
console.log(c.model); // Tesla
```

---

✅ **Summary:**

- If you `extend` another class → must call `super(...)` in your constructor before using `this`.
    
- If you don’t override → JS inserts a default constructor for you.
    
- If no `extends` → no need for `super`.
    