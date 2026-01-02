---
tags: 
 - js
 - class
 - inheritance
 - keyword
---

## 🔹 What is `super`?

The `super` keyword is used inside a subclass to **access the parent class’s constructor or methods**.

It helps you:

1. Call the parent’s **constructor** (`super(...)`)
    
2. Call a parent’s **method** inside an overriding method (`super.methodName(...)`)
    

---

## 🔹 1. Using `super` in constructors

When you extend a class, the child must call the parent’s constructor with `super(...)` **before using `this`**.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);        // calls Animal constructor
    this.breed = breed; // now safe to use "this"
  }
}

const d = new Dog("Rex", "Labrador");
console.log(d.name);  // Rex
console.log(d.breed); // Labrador
```

⚠️ If you don’t call `super()` in a child constructor, you’ll get a **ReferenceError** when trying to use `this`.

---

## 🔹 2. Using `super` in methods

If a child overrides a method but still wants to call the parent’s version, you use `super.methodName()`.

```js
class Animal {
  speak() {
    console.log("Generic animal sound");
  }
}

class Dog extends Animal {
  speak() {
    super.speak();   // call parent version
    console.log("Woof!");
  }
}

const d = new Dog();
d.speak();
// Generic animal sound
// Woof!
```

---

## 🔹 How `super` works internally

- `super` in a method is like saying:  
    “Go up the prototype chain one level, and call that method.”
    
- In a constructor, `super(...)` initializes the parent part of the object.
    
```ad-important
- `super` uses the **method definition from the parent**,
- but the **fields/properties it sees (`this.something`) belong to the child instance** that called it.
```

---

## 🔹 Prototype chain visualization with `super`

```
Dog.prototype.speak() {
  super.speak()  --> Animal.prototype.speak()
}
```

So `super` isn’t some magic — it’s just a **shortcut to parent prototype methods/constructor**.

---

## 🔹 Side note: Arrow functions and `super`

- Arrow functions in JavaScript **do not have their own `this`, `arguments`, or `super`**.
    
- They simply inherit those from the surrounding (lexical) scope.
    

This means if you try to use `super` inside an arrow function **defined directly in a class body**, it won’t work — because arrow functions don’t establish a `super` binding.

```js
class Animal {
  speak() {
    console.log("Generic sound");
  }
}

class Dog extends Animal {
  // ❌ arrow function cannot have super
  speak = () => {
    super.speak(); // SyntaxError: 'super' keyword unexpected
  };
}
```

✅ If you need to call `super`, you must use a **regular method**:

```js
class Dog extends Animal {
  speak() {
    super.speak(); // works fine
    console.log("Woof!");
  }
}
```

👉 In short:

- **Prototype methods** → can use `super`.
    
- **Class fields with arrow functions** → cannot use `super`.
    

---

## ❗Methods, not function properties

```js
let animal = {
  eat: function() {  // ❌ written with function expression
    // ...
  }
};

let rabbit = {
  __proto__: animal,
  eat: function() {  // ❌ function expression again
    super.eat();     // ERROR
  }
};

rabbit.eat(); // Error: no [[HomeObject]]
```

🔎 The problem:

- When you write `eat: function() { ... }`, JavaScript treats this as a **normal function property**, **not** as a method.
    
- **Normal function properties do not get `[[HomeObject]]`**, which is an internal slot required for `super` to work.
    
- So `super` has no idea where to look up the parent method, and it throws an error.
    

#### ✅ Correct way: method shorthand

If you write with shorthand syntax, it works:

```js
let animal = {
  eat() {   // ✅ method shorthand
    console.log("Animal eats");
  }
};

let rabbit = {
  __proto__: animal,
  eat() {   // ✅ method shorthand
    super.eat(); // works, [[HomeObject]] is set
  }
};

rabbit.eat(); // "Animal eats"
```

Here, `eat()` is recognized as a **method**, not just a function property.  
That automatically assigns a hidden `[[HomeObject]]`, which points to the object (`rabbit`), so `super` knows where to look.