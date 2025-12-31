---
tags: 
 - typescript
 - note
 - distinguish
---

### At a Glance

| **Feature**             | **Interface**                                       | **Intersection Type (&)**                        |
| ----------------------- | --------------------------------------------------- | ------------------------------------------------ |
| **Syntax**              | `interface A extends B { ... }`                     | `type A = B & C;`                                |
| **Goal**                | Defines the shape of an object.                     | Combines multiple types into one.                |
| **Declaration Merging** | **Yes** (merges automatically).                     | **No** (throws 'duplicate identifier' error).    |
| **Conflict Handling**   | Errors immediately if `extends` creates a conflict. | Merges conflicts (often resulting in `never`).   |
| **Performance**         | Faster (cached by name).                            | Slower (recursively computed).                   |
| **Best For**            | Public APIs, library definitions, object shapes.    | Utility types, combining existing types quickly. |

---

### 1. Syntax & Extensibility

The most common confusion is how they "combine" things.

Interface (using extends)

Interfaces are hierarchical. You strictly define that one interface inherits from another.

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Dog is { name: string; breed: string }
```

Intersection (using &)

Intersections are mathematical. You take two existing sets of data and "add" them together.

```ts
type Animal = {
  name: string;
};

type Dog = Animal & {
  breed: string;
};

// Dog is { name: string; breed: string }
```

### 2. Handling Conflicts (Crucial Difference)

This is where they behave very differently.

Interface:

If you try to extend an interface with a property that conflicts (has the wrong type), TypeScript will yell at you immediately.

```ts
interface A {
  id: number;
}

// ❌ Error: Interface 'B' incorrectly extends interface 'A'.
// Types of property 'id' are incompatible.
interface B extends A {
  id: string; 
}
```

Intersection:

Intersections are more passive. They will try to merge the types. If the types are incompatible (e.g., a value cannot be both a number and a string), the property becomes never.

```ts
type A = {
  id: number;
};

type B = A & {
  id: string;
};

// ✅ No Error during definition
// BUT 'id' is now type 'never' (number & string = impossible)
const item: B = {
  id: "123" // ❌ Error: Type 'string' is not assignable to type 'never'.
};
```

### 3. Declaration Merging (The "Magical" Feature)

Interfaces are "open". If you declare an `interface` twice, TypeScript merges them into a single definition. This is why libraries use interfaces—so you can add properties to them globally (like adding to the `Window` object).

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}

// User is now: { name: string; age: number }
```

**Intersection Types (via `type` alias)** are "closed". You cannot declare them twice.

```ts
type User = { name: string };
type User = { age: number }; // ❌ Error: Duplicate identifier 'User'.
```

### 4. Performance

- **Interfaces** are generally **faster** for the compiler. TypeScript caches interfaces by name.
    
- **Intersections** can be **slower** if used excessively in large recursion, as TypeScript has to compute the intersection every time it checks the type.
    

### Summary: Which one should I use?

1. **Use `interface` by default** for defining objects (e.g., React Props, database models, API responses). It provides better error messages and performance.
    
2. **Use `&` (Intersection)** when:
    
    - You are combining types you don't control (e.g., Third-party library type + your custom props).
        
    - You are creating quick, one-off types (e.g., `type Props = ButtonProps & { variant: 'primary' }`).
        
    - You need to combine things that aren't objects (e.g., primitives or Unions), though this is rare.
        

### Recommendation

If you are building an application, stick to **Interfaces** for your main data models and **Intersections** for quick utility combinations.