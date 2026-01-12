---
tags: 
 - typescript
 - union
 - note
---

## Accessing common properties on a **union type**

### Core rule

> You may only access **properties that exist on _all_ members** of a union.

---

## Basic example

```ts
type A = { id: number; name: string };
type B = { id: number; active: boolean };

type U = A | B;

u.id       // ✅ OK
u.name     // ❌ Error
u.active   // ❌ Error
```

Why:

- `id` exists on **both** `A` and `B`
    
- `name` and `active` do not
    

---

## How TypeScript decides

TypeScript computes the **intersection of keys**:

```ts
keyof (A | B)  // "id"
```

This is why property access is restricted.

---

## With functions

```ts
function printId(x: A | B) {
  console.log(x.id); // ✅ safe
}
```

---

## Narrowing to access specific properties

### 1️⃣ Using the `in` operator

```ts
function handle(x: A | B) {
  if ("name" in x) {
    x.name; // A
  } else {
    x.active; // B
  }
}
```

---

### 2️⃣ Discriminated unions (recommended)

```ts
type A = { kind: "a"; name: string };
type B = { kind: "b"; active: boolean };

function handle(x: A | B) {
  if (x.kind === "a") {
    x.name;
  } else {
    x.active;
  }
}
```

---

### 3️⃣ User-defined type guards

```ts
function isA(x: A | B): x is A {
  return "name" in x;
}
```

---

## Index access does NOT bypass safety

```ts
function get<K extends keyof (A | B)>(key: K) {
  // keyof (A | B) === "id"
}
```

You still only get common keys.

---

## Mental model

> **Union types represent “either this or that”,  
> so only shared structure is guaranteed.**

---

## Common misconception

```ts
type U = { a: number } | { b: number };
```

This does **not** mean:

```ts
{ a?: number; b?: number }
```

It means:

```ts
either { a } OR { b }
```

---

## Summary

| Scenario                   | Allowed |
| -------------------------- | ------- |
| Access shared property     | ✅      |
| Access non-shared property | ❌      |
| Narrow union first         | ✅      |
| Assume optionality         | ❌      |

---

## One-line takeaway

> **With union types, you can only access properties that exist on every union member unless you narrow first.**