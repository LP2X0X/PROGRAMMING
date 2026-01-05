---
tags: 
 - typescript
 - type
 - intersection
 - fundamental
---

An **intersection type** combines multiple types into **one type that must satisfy all of them simultaneously**.

```ts
type A = { a: number };
type B = { b: string };

type C = A & B;
// C must have both a and b
```

Think of `&` as **logical AND for types**.

---

## Core Semantics

- A value of type `A & B` **must conform to both `A` and `B`**
    
- Properties are **merged**
    
- Method signatures are **combined**
    
- Constraints become **stricter**, not looser
    

This is the opposite of a union (`|`), which is “either-or”.

---

## Property Merging Behavior

### Compatible properties

```ts
type X = { id: number };
type Y = { name: string };

type Z = X & Y;
// { id: number; name: string }
```

### Conflicting properties (important)

```ts
type A = { value: string };
type B = { value: number };

type C = A & B;
// value: never
```

If two properties share the same key but incompatible types, the result becomes `never`, making the type unusable.

```ad-note
We do not get warning here when we actually declare the intersection type. That's not the case for [[Type Interface|type interface]].
```

---

## Intersection with Function Types

Functions intersect by **overload-like behavior**.

```ts
type FnA = (x: number) => string;
type FnB = (x: string) => number;

type Fn = FnA & FnB;
```

This means `Fn` must support **both call signatures**.

---

## Common Use Cases

### 1. Object composition / mixins

```ts
type Timestamped = { createdAt: Date };
type Identified = { id: string };

type Entity = Timestamped & Identified;
```

Used heavily in:

- Domain models
    
- Entity systems
    
- Mixins
    

---

### 2. Extending existing types

```ts
type User = {
  name: string;
};

type UserWithRole = User & {
  role: "admin" | "user";
};
```

This is often preferred over inheritance for flexibility.

---

### 3. Combining generic constraints

```ts
function process<T extends A & B>(value: T) {
  // value has everything from A and B
}
```

This ensures the generic type satisfies **multiple structural requirements**.

---

### 4. Framework and library typing (very common)

You see intersections frequently in:

- React props
    
- Middleware contexts
    
- Configuration objects
    

```ts
type Props = BaseProps & WithTheme & WithAuth;
```

---

## Intersection vs Interface `extends`

```ts
interface A { a: number }
interface B { b: string }

interface C extends A, B {}
```

Equivalent to:

```ts
type C = A & B;
```

### Differences

| Aspect                | `&` (intersection) | `extends`    |
| --------------------- | ------------------ | ------------ |
| Works with primitives | Yes                | No           |
| Works with unions     | Yes                | No           |
| Declaration merging   | No                 | Yes          |
| Expressiveness        | Higher             | More limited |

Intersection types are **more general**.

---

## Intersection with Union Types (advanced)

```ts
type A = { a: number };
type B = { b: string };

type U = A | B;
type I = U & { common: boolean };
```

This distributes as:

```
(A & { common }) | (B & { common })
```

This pattern is used heavily in **discriminated unions** and **API modeling**.

---

## Pitfalls and Gotchas

### 1. Conflicting fields → `never`

Easy to create impossible types accidentally.

### 2. Harder error messages

Intersection-heavy types can produce verbose and confusing diagnostics.

### 3. Runtime does not exist

Intersection types are **compile-time only**. They do not merge objects at runtime.

```ts
const x = a as A & B; // no runtime check
```

---

## Mental Model

- Union (`|`) → _“one of these”_
    
- Intersection (`&`) → _“all of these”_
    
- More intersections = **stricter type**
    

---

## One-line Summary

> **Intersection types combine multiple types into a single type that must satisfy all constraints, enabling powerful composition and precise typing.**