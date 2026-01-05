---
tags: 
 - typescript
 - feature
---

In TypeScript, a **parameter property** is a shorthand feature that lets you **declare and initialize a class property directly in the constructor parameter list**.

It exists to reduce boilerplate when a constructor parameter is intended to become a class field.

---

## The Problem It Solves

Without parameter properties:

```ts
class User {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

This is repetitive: you declare properties, then assign them again.

---

## Parameter Property Syntax

You add an **access modifier** (or `readonly`) to a constructor parameter:

```ts
class User {
  constructor(
    public name: string,
    private age: number,
    readonly id: string
  ) {}
}
```

This single line per parameter does **three things**:

1. Declares a class property
    
2. Assigns the constructor argument to it
    
3. Applies the access modifier
    

Equivalent to:

```ts
class User {
  public name: string;
  private age: number;
  readonly id: string;

  constructor(name: string, age: number, id: string) {
    this.name = name;
    this.age = age;
    this.id = id;
  }
}
```

---

## Allowed Modifiers

A parameter becomes a parameter property only if it has one of these:

- `public`
    
- `private`
    
- `protected`
    
- `readonly`
    

Without one of these, it is just a normal parameter.

```ts
constructor(name: string) {} // NOT a parameter property
```

---

## Common Use Case

Parameter properties are most useful for **data-holder classes** and **dependency injection**.

Example (very common in frameworks):

```ts
class AuthService {}

class Controller {
  constructor(private authService: AuthService) {}
}
```

This:

- Declares `authService` as a private class field
    
- Automatically assigns it
    

---

## Access and Visibility

```ts
class Example {
  constructor(
    public a: number,
    private b: number,
    protected c: number,
    readonly d: number
  ) {}
}

const e = new Example(1, 2, 3, 4);

e.a; // OK
e.d; // OK (read-only)
e.b; // Error: private
e.c; // Error: protected
```

---

## Runtime Behavior (Important)

Parameter properties are **TypeScript-only** syntax.

At runtime (after compilation):

- They become **normal property assignments** in the constructor.
    
- There is no JavaScript concept called “parameter property”.
    

Emitted JavaScript (simplified):

```js
class User {
  constructor(name, age, id) {
    this.name = name;
    this.age = age;
    this.id = id;
  }
}
```

---

## Interaction with `readonly`

```ts
class Point {
  constructor(
    readonly x: number,
    readonly y: number
  ) {}
}

const p = new Point(1, 2);
p.x = 3; // Error (compile-time)
```

`readonly`:

- Prevents reassignment
    
- Does **not** make the object deeply immutable
    

---

## When _Not_ to Use Them

Avoid parameter properties when:

- Constructor logic is complex
    
- Property initialization needs validation or transformation
    
- You want clearer separation between API and internal state
    

Example to avoid:

```ts
constructor(private value: number) {
  if (value < 0) {
    throw new Error("Invalid value");
  }
}
```

Better:

```ts
private value: number;

constructor(value: number) {
  if (value < 0) {
    throw new Error("Invalid value");
  }
  this.value = value;
}
```

---

## Summary

- **Parameter properties** are a constructor shorthand
    
- Created by adding `public | private | protected | readonly`
    
- Reduce boilerplate for simple classes
    
- Compile to normal JavaScript assignments
    
- Best for simple data and DI patterns
    