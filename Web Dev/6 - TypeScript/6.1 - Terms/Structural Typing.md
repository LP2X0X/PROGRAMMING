---
tags: 
 - typescript
 - term
 - type
---

In TypeScript, **Structural Typing** is a way of relating types based solely on their members. This is often called "duck typing": _If it looks like a duck and quacks like a duck, it is a duck._

Unlike **Nominal Typing** (used in languages like Java, C#, or C++), where a class is only compatible if it explicitly inherits from another or shares the same name, TypeScript only cares if the "shape" of the objects matches.

---

## 1. The Core Principle: "Shape" over "Name"

In a structural system, two types are considered compatible if they have the same properties with the same types, regardless of how they were created.

```ts
interface Point {
  x: number;
  y: number;
}

class Vector2D {
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

// ✅ Works! Even though Vector2D doesn't "implement" Point
const v = new Vector2D(10, 20);
logPoint(v); 
```

---

## 2. Rule of Compatibility: "The Minimum Requirement"

For a type `A` to be compatible with type `B`, `A` must have **at least** all the properties required by `B`. It is allowed to have **more**, but it cannot have **less**.

### Example: Subtyping

```ts
interface Animal {
  name: string;
}

interface Dog {
  name: string;
  breed: string;
}

let animal: Animal;
let dog: Dog = { name: "Buddy", breed: "Golden Retriever" };

animal = dog; // ✅ OK: 'dog' has a 'name'.
// dog = animal; // ❌ Error: 'animal' is missing 'breed'.
```

You might have noticed that if you pass an object literal directly with extra properties, TypeScript often throws an error:

```ts
// ❌ Error: 'extra' does not exist in type '{ x: number; y: number }'
const s = new Shape({ x: 1, y: 2, extra: true });

// OK, no complain!
const shape = { x: 1, y: 2, extra: true };
const s = new Shape(shape);
```

**However**, when you pass a **variable** (like the `options` variable from your constructor), TypeScript relaxes this "excess property check." This allows for better composition and inheritance, as it assumes the variable might have originated from a context where those extra properties were necessary.

---

## 3. Strict Object Literal Checks

There is one major exception to structural typing: **Direct object literals**.

When you assign an object literal directly to a variable or pass it to a function, TypeScript applies "excess property checking." It assumes that if you are creating a fresh object, any extra properties are likely a mistake.

```ts
interface Person {
  name: string;
}

function greet(p: Person) {
  console.log(p.name);
}

// ❌ Error: 'age' is not part of 'Person'
greet({ name: "Alice", age: 30 }); 

// ✅ OK: Using a variable bypasses the strict check
const user = { name: "Alice", age: 30 };
greet(user); 
```

---

## 4. Function Compatibility

Structural typing also applies to functions. However, the rules feel "reversed" for arguments to ensure safety.

### Argument Counts

A function with **fewer** arguments is compatible with a function that expects **more** arguments. This is common in JavaScript (e.g., ignoring the `index` or `array` arguments in a `.forEach` loop).

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // ✅ OK: x can be used where y is expected (it just ignores the 2nd arg)
// x = y; // ❌ Error: y needs two arguments, but x only provides one
```

---

## 5. Class Compatibility

Classes are compared by their **instance members**. Static members and constructors do not affect compatibility.

```ts
class Car {
  engine: string = "V8";
  static wheels = 4;
}

class Boat {
  engine: string = "Outboard";
}

let vehicle: Car;
vehicle = new Boat(); // ✅ OK: Both have 'engine: string'
```

_Note: If a class has `private` or `protected` members, structural compatibility is broken. For private members, the properties must originate from the same class declaration to be compatible._

---

## 6. Summary: Why does this exist?

Structural typing is what makes TypeScript feel "natural" for JavaScript developers.

- **Flexibility:** You don't have to write `implements` for every interface.
    
- **Integration:** It works perfectly with existing JS libraries that weren't written with TypeScript in mind.
    
- **Composition:** It allows you to mix and match objects as long as they satisfy the required contract.
    

---

## References

https://www.typescriptlang.org/docs/handbook/type-compatibility.html