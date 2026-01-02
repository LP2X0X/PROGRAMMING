---
tags: 
 - typescript
 - class
 - constructor
---

## What Is a Constructor in TypeScript?

A constructor is a **special method** that runs **when a class instance is created** using `new`.

```ts
class User {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

const u = new User("Long");
```

Purpose:

- Initialize instance state
    
- Enforce required parameters
    
- Set up invariants
    

---

## Constructor Syntax Rules

- The method name **must be `constructor`**
    
- Only **one constructor** is allowed per class
    
- It has **no return type** (always returns the instance)
    

```ts
constructor(...) {
  // initialization only
}
```

---

## Parameter Properties (TypeScript-only)

TypeScript allows declaring and initializing properties directly in the constructor parameters.

```ts
class Point {
  constructor(
    public x: number,
    public y: number,
    private id: string
  ) {}
}
```

Equivalent to:

```ts
class Point {
  public x: number;
  public y: number;
  private id: string;

  constructor(x: number, y: number, id: string) {
    this.x = x;
    this.y = y;
    this.id = id;
  }
}
```

````ad-note
Constructor with optional parameter and default values for fields:
```ts
class CanvasNode {
  x: number; // You don't even need the type annotation here
  y: number;
  
  constructor(opt?: { x: number; y: number }) {
    this.x = opt?.x || 0;
    this.y = opt?.y || 0;
  }

  move(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}
```
````

---

## Access Modifiers in Constructors

Allowed modifiers:

- `public` (default)
    
- `private`
    
- `protected`
    
- `readonly`
    

```ts
class User {
  constructor(
    public readonly id: string,
    private password: string
  ) {}
}
```

- `id` cannot be reassigned
    
- `password` is inaccessible outside the class
    

---

## Constructors and Inheritance

### Base class

```ts
class Shape {
  constructor(public color: string) {}
}
```

### Derived class

```ts
class Circle extends Shape {
  constructor(color: string, public radius: number) {
    super(color); // required
  }
}
```

Rules:

- `super()` **must be called first**
    
- `this` cannot be used before `super()`
    

---

## Optional and Default Parameters

```ts
class User {
  constructor(
    public name: string,
    public age = 18,
    public email?: string
  ) {}
}
```

Fully type-checked at compile time.

---

## Overload Signatures (Advanced)

You can define multiple constructor signatures, but only **one implementation**.

```ts
class FileHandle {
  constructor(path: string);
  constructor(id: number);
  constructor(value: string | number) {
    // implementation
  }
}
```

---

## What Constructors Are _Not_

- ❌ Not static
    
- ❌ Cannot be async
    
- ❌ Cannot return custom values
    

```ts
constructor() {
  return {}; // ❌ ignored
}
```

---

## When the Constructor Runs

```ts
new MyClass();
```

Execution order:

1. Base class constructor (`super`)
    
2. Field initializers
    
3. Constructor body
    

---

## Summary

- The constructor initializes class instances
    
- TypeScript adds parameter properties and typing
    
- Only one constructor implementation is allowed
    
- `super()` is mandatory in derived classes
    
- Constructors enforce object correctness at creation time
    