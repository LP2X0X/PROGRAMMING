---
tags: 
 - typescript
 - type
 - technique
---

````ad-note
```ts
export const Logger = {
  log: (message: string) => {
    console.log(message);
  },
  info: (message: string) => {
    console.info(`INFO: ${message}`);
  },
  warn: (message: string) => {
    console.warn(`WARNING: ${message}`);
  },
  error: (message: string) => {
    console.error(`ERROR: ${message}`);
  },
};

export type Logger = typeof Logger;
```

```ts
import { Logger } from "./above";

const myApp = (logger: Logger) => {
  logger.log("Hello");
  logger.info("Hello");
  logger.warn("Hello");
  logger.error("Hello");
};

myApp(Logger);
```
````

You need a **type and a value with the same name** in TypeScript because the language deliberately models **JavaScript’s runtime** _and_ adds a **compile-time type system** on top of it. The two serve different purposes but are often naturally tied to the same concept.

---

## The Core Idea: Two Separate Worlds

TypeScript has **two namespaces**:

| Namespace       | Exists when  | Purpose                     |
| --------------- | ------------ | --------------------------- |
| **Value space** | Runtime      | Code that actually executes | 
| **Type space**  | Compile time | Static checking only        |

A single identifier can legally exist in **both**.

---

## Why This Is Useful (Not Redundant)

### 1. Model one concept in both worlds

Many real-world concepts need:

- **Runtime behavior** (functions, classes, objects)
    
- **Static guarantees** (types)
    

Using the same name keeps them aligned.

```ts
class User {
  name: string;
}
```

- `User` (value): constructor function at runtime
    
- `User` (type): instance shape at compile time
    

You get both from one declaration.

---

### 2. Zero duplication of domain concepts

Without dual names, you’d write:

```ts
class UserImpl {
  name: string;
}

type User = {
  name: string;
};
```

Now you must keep them in sync manually.

TypeScript avoids this by allowing **one symbol, two meanings**.

---

### 3. Enables powerful patterns (`typeof`, `keyof`)

Example:

```ts
const Roles = {
  Admin: "admin",
  User: "user",
} as const;

type Role = keyof typeof Roles;
```

Here:

- `Roles` (value): runtime object
    
- `typeof Roles` → type derived from real data
    

This ensures types **cannot drift** from values.

---

### 4. Supports gradual typing

TypeScript was designed to:

- Be valid JavaScript
    
- Add types _optionally_
    

Allowing value names to double as types lets you add typing **incrementally** without rewriting code structure.

---

## Concrete Use Cases

### Classes (most common)

```ts
class Service {}

function register(s: Service) {}
```

Same name, different roles.

---

### Enums (legacy but illustrative)

```ts
enum Status {
  Ready,
  Done,
}

let s: Status = Status.Ready;
```

---

### Const-object-as-enum pattern (modern)

```ts
const Status = {
  Ready: "ready",
  Done: "done",
} as const;

type Status = typeof Status[keyof typeof Status];
```

Same name, different spaces.

---

## When This Matters Most

- Library authors
    
- API boundaries
    
- Frameworks
    
- Type-safe configuration
    
- Erasable TypeScript (strip-types mode)
    

---

## Key Mental Model

> **Same name ≠ same thing**  
> It is one _concept_ represented in **two phases of the language**.

---

## One-Line Summary

> **TypeScript allows a name to exist as both a value and a type so runtime behavior and compile-time guarantees can stay aligned without duplication.**