---
tags: 
 - typescript
 - keyword
---

In TypeScript, the `implements` keyword is used to ensure that a **class** adheres to a specific **interface** or another class's structure. It acts as a strict contract: if a class says it `implements` an interface, it _must_ define all the properties and methods required by that interface.

Unlike `extends` (which inherits logic), `implements` only checks for **shape**.

---

### 1. Basic Implementation

When a class implements an interface, it is forced to provide implementations for everything defined in that interface.

```ts
interface Pingable {
  ping(): void;
  frequency: number;
}

// ✅ Correct implementation
class Sonar implements Pingable {
  frequency = 1000;
  
  ping() {
    console.log("Ping!");
  }
}

// ❌ Error: Class 'Ball' incorrectly implements interface 'Pingable'.
// Property 'ping' is missing.
class Ball implements Pingable {
  frequency = 10;
}
```

---

### 2. Implementing Multiple Interfaces

One of the biggest advantages of `implements` over `extends` is that a class can implement **multiple** interfaces at once, whereas it can only inherit from **one** parent class.

```ts
interface Flyable {
  fly(): void;
}

interface Swimmable {
  swim(): void;
}

class Duck implements Flyable, Swimmable {
  fly() {
    console.log("Duck is flying");
  }
  swim() {
    console.log("Duck is swimming");
  }
}
```

---

### 3. Important Caveat: It doesn't change the Class Type

A common mistake is thinking that `implements` will automatically type the class members. **It does not.** You still have to provide the types for the parameters and return values within the class itself.

```ts
interface Checker {
  check(name: string): boolean;
}

class NameChecker implements Checker {
  // ❌ Error: 'name' implicitly has 'any' type.
  // The interface doesn't "flow" types into the implementation.
  check(name) { 
    return name.length > 0;
  }
}
```

_You must explicitly type `name: string` in the class method to satisfy the compiler._

---

### 4. Implementing Classes

You can actually use a class name after the `implements` keyword. When you do this, TypeScript treats the class as an interface (using only its public shape, not its logic).

```ts
class Animal {
  move() {
    console.log("Moving...");
  }
}

// We are treating Animal as a SHAPE, not inheriting its code
class Robot implements Animal {
  move() {
    console.log("Robot moving on tracks");
  }
}
```

---

### Comparison: `extends` vs `implements`

| **Feature**      | **extends (Inheritance)**        | **implements (Contract)**                |
| ---------------- | -------------------------------- | ---------------------------------------- |
| **Relationship** | "is a" (Dog is an Animal)        | "acts as" (Clock acts as a Timer)        |
| **Logic**        | Inherits methods and properties. | Inherits **nothing** (must rewrite all). |
| **Count**        | Can only extend **one** class.   | Can implement **multiple** interfaces.   |
| **Purpose**      | Code reuse.                      | Type safety and consistency.             |

---

### Summary

- **Contract:** Use `implements` to force a class to have specific methods/properties.
    
- **Multiple:** Use it to satisfy multiple behaviors (e.g., `implements Logger, Disposable`).
    
- **Structure only:** It never copies code; it only checks that the code you wrote matches the blueprint.
    

---

## Reference

https://stackoverflow.com/questions/38834625/whats-the-difference-between-extends-and-implements-in-typescript