---
tags: 
 - typescript
 - class
 - overview
---

TypeScript classes add a layer of type safety and object-oriented features on top of standard JavaScript classes. They allow you to control exactly how properties are accessed, modified, and inherited.

Here is a detailed overview of the key features that TypeScript adds to classes.

---

### 1. Basic Structure & Type Annotation

In TypeScript, you must declare the properties of a class and their types _before_ you use them in the constructor (unless you use "Parameter Properties," covered later).

```ts
class User {
  // 1. Declare properties and types upfront
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  // Method
  greet(): string {
    return `Hello, I am ${this.name}`;
  }
}
```

````ad-note
Even though class is a value (function), it can still act like a type:
```ts
import { describe, expect, it } from "vitest";

class CanvasNode {
  x = 0;
  y = 0;

  move(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const positionFromCanvasNode = (node: CanvasNode) => {
  return {
    x: node.x,
    y: node.y,
  };
};
```
````

### 2. Access Modifiers (Visibility)

This is one of the most useful features TS adds. It controls who can see or use a property.

| **Modifier**           | **Where it is accessible**                                       |
| ---------------------- | ---------------------------------------------------------------- |
| **`public`** (Default) | Anywhere (inside the class, instances, subclasses).              |
| **`private`**          | **Only** inside the specific class it is defined in.             | 
| **`protected`**        | Inside the class **and** any class that extends it (subclasses). |

```ts
class BankAccount {
  public id: string;           // Accessible anywhere
  private balance: number;     // Only accessible inside BankAccount
  protected type: string;      // Accessible in BankAccount and subclasses

  constructor(id: string, balance: number) {
    this.id = id;
    this.balance = balance;
    this.type = "checking";
  }

  public getBalance() {
    return this.balance; // ✅ OK: Private is accessible here
  }
}

const account = new BankAccount("123", 1000);
// console.log(account.balance); // ❌ Error: 'balance' is private
```

> **Note:** TypeScript's `private` is a "soft" privacy (checked at compile time). If you need true runtime privacy, use the JavaScript native hash syntax (e.g., `#balance`).

### 3. Parameter Properties (Shorthand)

TypeScript offers a shorthand to declare and initialize properties in one step by adding an access modifier directly in the constructor arguments. This is very common in frameworks like Angular or NestJS.

**The Long Way:**

```ts
class Car {
  make: string;
  constructor(make: string) {
    this.make = make;
  }
}
```

**The Short Way (Parameter Properties):**

```ts
class Car {
  // TypeScript automatically creates 'this.make' and assigns the value
  constructor(public make: string) {} 
}
```

### 4. `readonly` Modifier

The `readonly` keyword allows a property to be set **only** in the constructor. After that, it is immutable.

```ts
class Book {
  readonly title: string;

  constructor(title: string) {
    this.title = title;
  }

  changeTitle() {
    // this.title = "New Title"; // ❌ Error: Cannot assign to 'title' because it is a read-only property.
  }
}
```

### 5. Getters and Setters (`get` / `set`)

These allow you to intercept access to a property. This is great for validation or computed values.

```ts
class Employee {
  private _fullName: string = "";

  get fullName(): string {
    return this._fullName;
  }

  set fullName(newName: string) {
    if (newName.length < 3) {
      throw new Error("Name is too short!");
    }
    this._fullName = newName;
  }
}

const emp = new Employee();
emp.fullName = "John Doe"; // Calls the setter
console.log(emp.fullName); // Calls the getter
```

### 6. Inheritance (`extends`)

Classes can inherit from other classes using `extends`. You use `super()` to call the parent constructor or parent methods.

```ts
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  bark() {
    console.log("Woof!");
  }

  // Overriding a parent method
  move() {
    super.move(); // Call original method
    console.log("...and running fast.");
  }
}

const dog = new Dog();
dog.move(); // Output: "Moving along!" then "...and running fast."
```

### 7. Abstract Classes

An `abstract` class cannot be instantiated directly (you can't say `new AbstractClass()`). It serves as a blueprint for other classes.

If a method is marked `abstract`, it has **no implementation** inside the abstract class; the child class **must** implement it.

```ts
abstract class Shape {
  // Abstract method: Must be implemented by children
  abstract getArea(): number;

  // Regular method: Inherited as-is
  printArea() {
    console.log("Area is: " + this.getArea());
  }
}

class Circle extends Shape {
  constructor(public radius: number) { super(); }

  // We are FORCED to implement this
  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

// const s = new Shape(); // ❌ Error: Cannot create an instance of an abstract class.
const c = new Circle(5);  // ✅ OK
```

### 8. Static Members

`static` properties or methods belong to the **class itself**, not the instance (the object created from the class).

```ts
class MathUtil {
  static PI = 3.14;

  static calculateCircumference(radius: number) {
    return 2 * MathUtil.PI * radius;
  }
}

console.log(MathUtil.PI); // Accessed directly on the class
// const m = new MathUtil();
// m.PI; // ❌ Error: Property 'PI' does not exist on type 'MathUtil'.
```

---

### Summary Checklist

| **Feature**      | **Keyword**                        | **Description**                                               |
| ---------------- | ---------------------------------- | ------------------------------------------------------------- |
| **Visibility**   | `public` / `private` / `protected` | Controls who can see the property.                            |
| **Immutability** | `readonly`                         | Property cannot be changed after initialization.              |
| **Shorthand**    | `constructor(public x: string)`    | Creates and assigns property automatically.                   |
| **Blueprint**    | `abstract`                         | Class cannot be instantiated; forces structure on subclasses. |
| **Class-Level**  | `static`                           | Belongs to the class definition, not specific objects.        |