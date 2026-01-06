---
tags: 
 - typescript
 - feature
---

### 1. What is an Enum?

Enums allow you to define a set of named constants. They make code more readable and less error-prone by replacing "magic numbers" or strings with descriptive names. Unlike most TypeScript features which disappear after compilation, **Enums actually generate real JavaScript code** (objects) at runtime.

```ad-note
[[erasableSyntaxOnly]] must be false to use enum.
```

```ad-note
**Enums break TypeScript’s [[Structural Typing|structural typing]] rule** because they introduce **nominal identity**: even if two enums have identical members and values, they are treated as **distinct, non-assignable types**, unlike structurally identical object or union types.
```

---

### 2. Numeric Enums

By default, enums are numeric. They auto-increment starting from `0`.

#### Default Behavior

```ts
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

const move = Direction.Up; // Value is 0
```

#### Custom Initialization

You can set the starting value, and subsequent members will increment by 1.

```ts
enum StatusCode {
  Success = 200,
  NotFound = 404,
  ServerError = 500
}

enum Step {
  First = 1,
  Second, // becomes 2
  Third   // becomes 3
}
```

#### Reverse Mapping

A unique feature of **Numeric Enums** is that you can access the member name from its value.

```ts
enum Size {
  Small = 1,
  Medium = 2
}

console.log(Size.Small);    // Output: 1
console.log(Size[1]);       // Output: "Small" (Reverse Mapping)
```

_Note: This works because TS generates an object that looks like `{ Small: 1, 1: "Small" }`._

---

### 3. String Enums

String enums do **not** have auto-incrementing behavior; you must initialize each member with a string literal.

```ts
enum Role {
  Admin = "ADMIN",
  User = "USER",
  Guest = "GUEST"
}
```

- **Pros:** Better debuggability (you see "ADMIN" in logs instead of `0`).
    
- **Cons:** **No Reverse Mapping**. You cannot do `Role["ADMIN"]` to get the key `Admin`.
    

---

### 4. Heterogeneous Enums

You can mix strings and numbers, but this is generally **discouraged** as it causes confusion.

```ts
enum Mixed {
  No = 0,
  Yes = "YES"
}
```

---

### 5. `const` Enums (Optimization)

If you do not need the enum object at runtime (e.g., you don't need to iterate over it) and want to optimize performance/bundle size, use `const enum`.

**TypeScript:**

```ts
const enum Direction {
  Up,
  Down
}
let move = Direction.Up;
```

**Compiled JavaScript:**

```js
let move = 0; /* Up */
```

_The enum definition is completely removed, and the value is inlined._

---

### 6. Enums vs. Union Types

This is a common debate in the TS community.

#### The "Union Type" Alternative

Many modern TS developers prefer String Literal Unions over String Enums.

```ts
// Enum approach
enum Alignment {
  Left = "left",
  Right = "right"
}

// Union Type approach
type AlignmentType = "left" | "right";
```

| **Feature**               | **Enum**                    | **Union Type**                      |
| ------------------------- | --------------------------- | ----------------------------------- |
| **Runtime Code**          | Generates an Object         | Disappears (Zero overhead)          |
| **Nominal vs Structural** | **Nominal** (Specific Type) | **Structural** (Matches value)      |
| **Iteration**             | Easy to iterate keys/values | Hard to iterate (needs extra array) |

**When to use which?**

- **Use Enums if:** You need to iterate over the values (e.g., populating a dropdown list) or you prefer the strict nominal typing.
    
- **Use Unions if:** You want zero runtime overhead and simple string matching.
    

---

### 7. Objects as Enums (`as const`)

A modern pattern that bridges the gap between Enums and Objects is using `as const`.

```ts
const Colors = {
  Red: "#FF0000",
  Green: "#00FF00",
  Blue: "#0000FF"
} as const; // Makes properties readonly and literal types

// Extract the values as a type
type Color = typeof Colors[keyof typeof Colors]; // "#FF0000" | "#00FF00" | "#0000FF"

const myColor: Color = Colors.Red;
```

This keeps the JavaScript object (so you can iterate) but provides strict typing like an Enum.

---

### Summary Checklist

- [ ] **Numeric Enums:** Good for flags or math-heavy constants. Supports reverse mapping.
    
- [ ] **String Enums:** Good for readability (logs/API values). No reverse mapping.
    
- [ ] **Const Enums:** Use for internal constants to reduce bundle size (values inlined).
    
- [ ] **Union Types:** Use for simple state management (e.g., `status: 'loading' | 'success'`) to avoid runtime bloat.
    