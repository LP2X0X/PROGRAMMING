---
tags: 
 - typescript
 - term
---

### What an Interface Is

An `interface` defines the **shape of an object**. It describes what properties and methods an object must have, without providing any implementation.

Used primarily for:

- Object structures
    
- Function contracts
    
- Class contracts
    
- Public APIs
    

---

### Basic Syntax

```ts
interface User {
  id: number;
  name: string;
  isAdmin: boolean;
}
```

Any object assigned to `User` must match this structure.

---

### Structural Typing

TypeScript uses **structural typing**, not nominal typing.

```ts
const user = {
  id: 1,
  name: "Long",
  isAdmin: false,
};

let u: User = user; // valid
```

The object does not need to explicitly declare it implements the interface.

---

### Optional Properties

```ts
interface User {
  id: number;
  name: string;
  age?: number;
}
```

`?` means the property may be absent.

---

### Readonly Properties

```ts
interface User {
  readonly id: number;
  name: string;
}
```

Prevents reassignment after initialization.

---

### Function Signatures

```ts
interface Formatter {
  (value: string): string;
}
```

Defines callable shapes.

---

### Method Definitions

```ts
interface User {
  login(): void;
  logout(): void;
}
```

Methods are part of the contract.

---

### Extending Interfaces

```ts
interface Admin extends User {
  permissions: string[];
}
```

Supports inheritance-like composition.

---

### Interface vs Type Alias (Key Differences)

- Interfaces are **extendable**
    
- Interfaces support **declaration merging**
    
- Type aliases can represent unions and primitives
    

Use interfaces for:

- Object shapes
    
- Public contracts
    
- Library APIs
    

---

### Declaration Merging

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}
```

Merged into:

```ts
interface User {
  name: string;
  age: number;
}
```

This is unique to interfaces.

---

### Implementing Interfaces in Classes

```ts
class Person implements User {
  constructor(
    public id: number,
    public name: string,
    public isAdmin: boolean
  ) {}
}
```

Ensures the class follows the interface contract.

---

### Index Signatures

```ts
interface Dictionary {
  [key: string]: string;
}
```

Allows flexible property names with fixed value types.

---

### When to Use Interfaces

- Defining object contracts
    
- Designing scalable APIs
    
- Modeling data structures
    
- Enforcing consistency across codebases
    

---

### Key Takeaway

Interfaces define **what a structure must look like**, not **how it works**. They are compile-time only and disappear entirely at runtime.