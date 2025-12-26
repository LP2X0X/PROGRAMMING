---
tags: 
 - typescript
 - term
---

In TypeScript, a **type argument** is the concrete type you supply to a **generic** type, function, interface, or class to specify how it should be typed.

### Core idea

- **Generic**: defines a placeholder type (e.g., `T`)
    
- **Type argument**: the actual type you pass in place of that placeholder (e.g., `number`, `string`)
    

### Basic examples

**Generic function**

```ts
function identity<T>(value: T): T {
  return value;
}

identity<number>(42);   // number is the type argument
identity("hello");      // inferred type argument: string
```

**Generic interface**

```ts
interface ApiResponse<T> {
  data: T;
  error?: string;
}

const res: ApiResponse<User> = {
  data: { id: 1, name: "Alice" }
};
```

**Generic class**

```ts
class Box<T> {
  constructor(public value: T) {}
}

const box = new Box<string>("text");
```

### Multiple type arguments

```ts
type Pair<K, V> = {
  key: K;
  value: V;
};

const p: Pair<string, number> = { key: "age", value: 30 };
```

### Constraints on type arguments

You can restrict what types are allowed.

```ts
function logLength<T extends { length: number }>(value: T): void {
  console.log(value.length);
}

logLength("abc");   // OK
logLength([1, 2]);  // OK
// logLength(123);  // Error
```

### Defaults for type arguments

```ts
type Result<T = string> = {
  value: T;
};

const a: Result = { value: "ok" };      // uses default: string
const b: Result<number> = { value: 1 }; // overrides default
```

### When you need to write them explicitly

- When inference fails or is ambiguous
    
- When you want stricter or clearer typing at the call site
    
- In public APIs or libraries for readability
    

In short: **type arguments specialize generics**, turning reusable, abstract types into concrete, type-safe ones.