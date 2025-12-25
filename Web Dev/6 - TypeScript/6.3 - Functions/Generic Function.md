---
tags: 
 - typescript
 - function
 - generic
---

### What a generic function is 🔍

A **generic function** is a function that:

- Works with **multiple types**
    
- While **preserving type information**
    
- Without using `any`
    

```ts
function identity<T>(value: T): T {
  return value;
}
```

🧠 `T` is a **type parameter** — a placeholder for a real type.

---

## Why generics exist 🎯

Without generics:

```ts
function identity(value: any): any {
  return value;
}
```

Problems:

- Type information is lost
    
- No compile-time safety
    
- No autocomplete
    

With generics:

```ts
const x = identity(10);     // number
const y = identity("hi");  // string
```

✔ Full type safety  
✔ No duplication

---

## Basic syntax 🧠

```ts
function fn<T>(arg: T): T
```

- `<T>` → declares a type parameter
    
- `T` can be used in:
    
    - parameters
        
    - return types
        
    - constraints
        

---

## Type inference (most important feature) ✨

```ts
identity(123);
```

TypeScript infers:

```ts
T = number
```

You usually **do not need to write `<number>` explicitly**.

---

### Explicit generic arguments (rare but useful)

```ts
identity<string>("hello");
```

Used when:

- inference fails
    
- you want to force a specific type
    

---

## Multiple type parameters 🔁

```ts
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}
```

```ts
const p = pair("id", 123);
// [string, number]
```

---

## Generic constraints 🔒

### Using `extends`

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```

✔ `Type` must have `length`

---

### Using `keyof`

```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

✔ `key` must be a valid property of `obj`

---

## Generics vs unions 🆚

### Generic (preserves relationship)

```ts
function wrap<T>(x: T): T[] {
  return [x];
}
```

### Union (loses relationship)

```ts
function wrap(x: string | number): (string | number)[] {
  return [x];
}
```

Generics are preferred when **input and output are related**.

---

## Generic arrow functions ⚠️

```ts
const identity = <T,>(value: T): T => value;
```

The trailing comma avoids JSX parsing issues.

---

## Generic defaults 🎯

```ts
function createMap<K, V = string>() {
  return new Map<K, V>();
}
```

```ts
const m = createMap<number>();
// Map<number, string>
```

---

## Generics with async functions ⚙️

```ts
async function fetchData<T>(): Promise<T> {
  ...
}
```

Usage:

```ts
const user = await fetchData<User>();
```

---

## Generics with constructors 🏗️

```ts
function create<T>(ctor: new () => T): T {
  return new ctor();
}
```

This pattern appears frequently in DI and factories.

---

## Common mistakes 🚧

### ❌ Using generics when not needed

```ts
function log<T>(value: T): void {
  console.log(value);
}
```

`T` adds no value here.

---

### ❌ Over-constraining generics

```ts
function fn<T extends string>(x: T): T
```

Often unnecessary unless enforcing rules.

---

### ❌ Confusing generics with `any`

```ts
function bad<T>(x: T): string {
  return x; // ❌
}
```

---

## When to use generic functions 🎯

Use generics when:

- Input and output types are related
    
- You want reusable logic
    
- You want full type inference
    
- `any` would otherwise be used
    

---

## Mental model 🧠

- Generics are **type variables**
    
- They are resolved at **call time**
    
- They disappear at runtime
    
- They preserve relationships between types
    

---

## Summary 📝

- Generic functions enable reusable, type-safe logic
    
- `<T>` is a placeholder, not a real type
    
- Inference is usually automatic
    
- Constraints refine what `T` can be
    
- Common in libraries and frameworks
    
