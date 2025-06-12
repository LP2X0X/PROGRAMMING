---
tags: js, term, fundamental
---

In JavaScript, method override and method shadowing both refer to how methods can be redefined in derived or child objects/classes, but they happen in slightly different contexts.

### **1. Method Override**

This happens in class inheritance. A subclass redefines a method from its parent class.

```js
class Animal {
  speak() {
    console.log("Animal speaks");
  }
}

class Dog extends Animal {
  speak() {
    console.log("Dog barks");
  }
}

const myDog = new Dog();
myDog.speak(); // Outputs: Dog barks
```

- Here, Dog overrides the speak() method from Animal.
- If you call speak() on an instance of Dog, the Dog version is used.
- You can still call the parent version using super:

```js
class Dog extends Animal {
  speak() {
    super.speak(); // Animal speaks
    console.log("Dog barks");
  }
}
```

### **2. Method Shadowing**

This happens when a method or property in a child object hides one in the parent, without traditional class-based inheritance—often in object literals or prototypal inheritance.

```js
const parent = {
  greet() {
    console.log("Hello from parent");
  }
};

const child = Object.create(parent);
child.greet = function() {
  console.log("Hello from child");
};

child.greet(); // Outputs: Hello from child
```

- The child object “shadows” the greet() method from the parent.
- You can still access the parent version manually:

```js
Object.getPrototypeOf(child).greet.call(child); // Hello from parent
```

### **Summary**

|**Concept**|**Context**|**Access to Parent?**|
|---|---|---|
|Override|Classes|Yes, via super|
|Shadowing|Object prototypes|Yes, via prototype|

Both are ways of redefining behavior, but “override” is class-focused and more formal, while “shadowing” happens in object composition or prototype chains.
