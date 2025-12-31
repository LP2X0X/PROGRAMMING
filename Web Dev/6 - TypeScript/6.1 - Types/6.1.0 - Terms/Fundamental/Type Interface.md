---
tags: 
 - typescript
 - type
 - interface
 - fundamental
---

An **`interface`** defines the **shape of an object**: what properties it has, their types, and the signatures of methods it exposes.

```ts
interface User {
  id: number;
  name: string;
}
```

Interfaces are **purely compile-time constructs**. They do not exist at runtime.

---

## What interfaces are used for

Interfaces are primarily used to:

- Describe object structures
    
- Define contracts between components
    
- Type class implementations
    
- Model API data
    
- Enable safe polymorphism
    

---

## Basic Syntax

```ts
interface Product {
  id: string;
  price: number;
  inStock: boolean;
}
```

---

## Optional properties

```ts
interface Config {
  url: string;
  timeout?: number;
}
```

`?` means the property may be omitted.

---

## Readonly properties

```ts
interface Point {
  readonly x: number;
  readonly y: number;
}
```

Prevents reassignment after initialization.

---

## Function signatures in interfaces

```ts
interface Comparator {
  (a: number, b: number): number;
}
```

Defines the callable shape of a function.

---

## Methods vs properties

```ts
interface Service {
  start(): void;
  stop: () => void;
}
```

Both are valid; method syntax supports `this` typing more naturally.

---

## Index signatures

```ts
interface StringMap {
  [key: string]: string;
}
```

Used when property names are not known ahead of time.

---

## Extending interfaces

```ts
interface Base {
  id: string;
}

interface User extends Base {
  name: string;
}
```

Interfaces support **multiple inheritance**.

```ts
interface A { a: number }
interface B { b: number }

interface C extends A, B {}
```

---

## Interfaces vs type aliases

| Aspect                    | `interface` | `type`    |
| ------------------------- | ----------- | --------- |
| Object shapes             | Yes         | Yes       |
| Primitives                | No          | Yes       |
| Unions / intersections    | No          | Yes       |
| Declaration merging       | Yes         | No        |
| Extends                   | Yes         | Via `&`   |
| Preferred for public APIs | Yes         | Sometimes |

---

## Declaration merging (important feature)

```ts
interface Window {
  title: string;
}

interface Window {
  isOpen: boolean;
}
```

Merged into:

```ts
interface Window {
  title: string;
  isOpen: boolean;
}
```

This is heavily used in:

- DOM typings
    
- Library augmentation
    
- Global namespace extension
    

---

## Implementing interfaces with classes

```ts
interface Flyable {
  fly(): void;
}

class Bird implements Flyable {
  fly() {}
}
```

Classes must fully satisfy the interface contract.

---

## Interfaces and generics

```ts
interface ApiResponse<T> {
  data: T;
  error?: string;
}
```

Common in APIs and libraries.

---

## Interfaces vs runtime behavior

Interfaces:

- Are erased at compile time
    
- Do not enforce runtime validation
    
- Require explicit checks if used with external data
    

---

## When to prefer `interface`

Use `interface` when:

- Modeling object shapes
    
- Designing public or shared APIs
    
- You want declaration merging
    
- You expect others to extend your types
    

Use `type` when:

- You need unions or intersections
    
- You are working with primitives or tuples
    
- You need advanced type composition
    

---

## Mental model

> An `interface` is a **structural contract**, not a value or a class.

---

## One-line summary

> **`interface` defines the shape of objects and contracts in TypeScript, enabling safe composition, extension, and implementation without runtime cost.**

If you want, I can next explain:

- `interface` vs `type` in React props
    
- Why interfaces are structural (not nominal)
    
- How declaration merging works in real libraries