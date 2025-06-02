---
tags: js, keyword, fundamental
---

In JavaScript, the `super` keyword is used in [[Web Dev/3 - JavaScript Crash Course/1 - Basic/Classes/Class|classes]] to refer to the parent class. It allows you to access and call functions (including the constructor) of a parent class from a subclass.

---

### 📌 When to Use `super`?

The `super` keyword is typically used in:

1. **Subclass constructors** – to call the constructor of the parent class.
2. **Subclass methods** – to call a method defined in the parent class.

---

### 🧱 Syntax and Examples

#### 1. Calling the parent constructor with `super()`

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a noise.`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Call the parent class constructor
    this.breed = breed;
  }

  info() {
    console.log(`${this.name} is a ${this.breed}`);
  }
}

const d = new Dog('Buddy', 'Golden Retriever');
d.speak(); // Buddy makes a noise.
d.info();  // Buddy is a Golden Retriever
```

✅ `super(name)` is used to pass the `name` to the parent class constructor.

---

#### 2. Calling parent methods with `super.methodName()`

```javascript
class Animal {
  speak() {
    console.log("Animal sound");
  }
}

class Dog extends Animal {
  speak() {
    super.speak(); // Call parent method
    console.log("Woof!");
  }
}

const dog = new Dog();
dog.speak();
// Output:
// Animal sound
// Woof!
```

✅ `super.speak()` calls the `speak` method from the parent `Animal` class.

---

### ⚠️ Important Rules

- You must call `super()` before using `this` in a subclass constructor.
  
  ```javascript
  class Dog extends Animal {
    constructor(name) {
      // this.name = name; ❌ Error: Must call super before accessing 'this'
      super(name);         // ✅ Correct
    }
  }
  ```

- `super` can only be used in class constructors or methods inside classes using `extends`.

---

### Bonus: `super` in Static Methods

You can also use `super` in static methods to call static methods of the parent class.

```javascript
class Parent {
  static hello() {
    return "Hello from Parent";
  }
}

class Child extends Parent {
  static hello() {
    return super.hello() + " and Hello from Child";
  }
}

console.log(Child.hello()); 
// Output: Hello from Parent and Hello from Child
```

---

### Summary

| Use Case             | Syntax              | Purpose                                 |
|----------------------|---------------------|------------------------------------------|
| Call parent constructor | `super(args)`       | Initialize parent class part             |
| Call parent method   | `super.method()`    | Reuse or extend parent method logic      |
| In static methods    | `super.method()`    | Access static methods from parent class  |
