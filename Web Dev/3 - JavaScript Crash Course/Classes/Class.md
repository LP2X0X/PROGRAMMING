---
tags: js, term, fundamental
---

In JavaScript, a class is a blueprint for creating objects with pre-defined properties and methods. Classes were introduced in ECMAScript 2015 (also known as ES6) and provide a cleaner and more intuitive syntax for creating constructor functions and inheritance compared to the older prototype-based approach.

### Basic Syntax of a Class

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // Method
  greet() {
    console.log(`Hello, my name is ${this.name} and I am ${this.age} years old.`);
  }
}

// Create an instance of the class
const person1 = new Person("Alice", 30);
person1.greet(); // Output: Hello, my name is Alice and I am 30 years old.
```

### Key Features

1. **Constructor Method**
   - The `constructor()` method is automatically called when a new instance of the class is created using `new`.

2. **Methods**
   - You can define methods directly inside the class body. They become part of the prototype of the class.

3. **Inheritance**
   - A class can extend another class using the `extends` keyword and can use the `super` function to call the parent’s constructor or methods.

```javascript
class Employee extends Person {
  constructor(name, age, jobTitle) {
    super(name, age); // Call the constructor of the parent class
    this.jobTitle = jobTitle;
  }

  describeJob() {
    console.log(`${this.name} works as a ${this.jobTitle}.`);
  }
}

const emp1 = new Employee("Bob", 25, "Software Engineer");
emp1.greet();       // Hello, my name is Bob and I am 25 years old.
emp1.describeJob(); // Bob works as a Software Engineer.
```

4. **Getters and Setters**

```javascript
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  get area() {
    return this.width * this.height;
  }

  set dimensions([width, height]) {
    this.width = width;
    this.height = height;
  }
}

const rect = new Rectangle(10, 5);
console.log(rect.area); // 50
rect.dimensions = [20, 10];
console.log(rect.area); // 200
```

5. **Static Methods**
   - Static methods belong to the class itself, not instances.

```javascript
class MathUtils {
  static add(a, b) {
    return a + b;
  }
}

console.log(MathUtils.add(5, 3)); // 8
```

### Summary

- JavaScript classes provide a cleaner and modern way to work with objects and inheritance.
- Use `constructor()` for initialization.
- Use `extends` for inheritance.
- Use `super` to access parent class constructors or methods.
- Use static methods for utility functions that don't require an instance.
