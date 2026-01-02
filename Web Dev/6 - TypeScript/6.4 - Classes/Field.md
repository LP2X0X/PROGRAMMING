---
tags: 
 - typescript
 - class
 - field
---

A **field** declaration creates a public writeable property on a class:

```ts
class Point {
  x: number;
  y: number;
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```
 
As with other locations, the type annotation is optional, but will be an implicit any if not specified.

Fields can also have initializers; these will run automatically when the class is instantiated:

```ts
class Point {
  x = 0;
  y = 0;
}
 
const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

```ad-note
Just like with const, let, and var, the initializer of a class property will be used to infer its type.
```

```ad-warning
Don't initialize fields in method. TypeScript does not analyze methods you invoke from the constructor to detect initializations, because a derived class might override those methods and fail to initialize the members.
```

---

## readonly Field

Fields may be prefixed with the readonly modifier. This prevents assignments to the field *outside of the constructor*.

```ts
class Greeter {
  readonly name: string = "world";
 
  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }
 
  err() {
    // Cannot assign to 'name' because it is a read-only property.
    this.name = "not ok";
  }
}

const g = new Greeter();
// Cannot assign to 'name' because it is a read-only property.
g.name = "also not ok";

```
