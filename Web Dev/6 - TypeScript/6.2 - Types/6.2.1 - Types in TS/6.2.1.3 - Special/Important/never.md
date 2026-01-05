---
tags: 
 - typescript
 - type
 - special
---

The `never` type represents values which are _never_ observed. In a return type, this means that the function throws an exception or terminates execution of the program.

`never` also appears when TypeScript determines there’s nothing left in a union.

```ad-note
never sits at the bottom of the types hierarchy in TypeScript.
```

![[067-introduction-to-never.explainer.svg|center|700]]
 <figcaption align="center"><strong>Figure:</strong> Assignability chart where never can be assign to all types but not vice versa. </figcaption>

#### Example
Use case: Exhaustive Checks

```ts
function foo(x: string | number): boolean {
  if (typeof x === "string") {
    return true;
  } else if (typeof x === "number") {
    return false;
  }

  // Without a never type we would error :
  // - Not all code paths return a value (strict null checks)
  // - Or Unreachable code detected
  // But because TypeScript understands that `fail` function returns `never`
  // It can allow you to call it as you might be using it for runtime safety / exhaustive checks.
  return fail("Unexhaustive!");
}

function fail(message: string): never { throw new Error(message); }
```

```ts
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

```ts
import { Equal, Expect } from "@total-typescript/helpers";

// If we return undefined, id will have type string | undefined
// const throwError = (message: string): undefined => {
//   throw new Error(message);
// };

// Now, id will can only be of type string
const throwError = (message: string): never => {
  throw new Error(message);
};

const handleSearchParams = (params: { id?: string }) => {
  const id = params.id || throwError("No id provided"); 

  type test = Expect<Equal<typeof id, string>>;

  return id;
};
```

#### Main situations where `never` appears automatically

| Situation                                 | Why it’s `never`                                      |
|-------------------------------------------|-------------------------------------------------------|
| Function that **never returns**           | It throws or loops forever                            |
| Infinite loop                             | No way to reach the end                               |
| Exhaustive `switch` / `if` checks         | All possible cases of a union are handled → impossible branch is `never` |
| Unreachable code after `return`/`throw`   | TypeScript narrows to `never`                         |

#### Why do we need the `never` type?

Because TypeScript needs a way to **catch impossible or missing code paths** — especially in:

1. **Exhaustive checks**
    
2. **Functions that never return**
    
3. **Impossible type situations**
    

#### Key properties of `never`

- `never` is a **subtype** of every type (bottom type) → you can assign `never` to anything.
  ```ts
  const x: never = …;
  const s: string = x;   // OK
  const n: number = x;   // OK
  ```
- Nothing (except `never` itself) is assignable to `never`.
  ```ts
  let x: never;
  x = "hello";   // Error
  ```
- Arrays of `never` are empty (`never[]` has length 0).
- Useful for **exhaustiveness checking** (the classic use case).

#### Practical patterns

```ts
// 1. Exhaustiveness check (most common)
type Shape = Circle | Square | Triangle;

function area(s: Shape): number {
  switch (s.kind) {
    case "circle":  return Math.PI * s.r ** 2;
    case "square":  return s.side ** 2;
    case "triangle": return s.base * s.height / 2;
    // Add new shape? You’ll get a compile error here:
    default:        exhaustive(s);  // s has type never
  }
}

function exhaustive(x: never): never {
  throw new Error("impossible: " + x);
}
```

```ts
// 2. Mark intentionally unreachable branches
if (false) {
  console.log("this never runs");  // any code here is typed as never
}
```

#### TL;DR

- `never` = “this value can never happen”.
- Use it for functions that don’t return, exhaustive checks, and to get compiler errors when you forget to handle a new union case.
- It’s the safest way to tell TypeScript “trust me, this branch is impossible”.

---

## Other explanation

The Typescript compiler assigns types to all your variables/properties. These types can be either the typescript defined types like `number` and `string`, or user defined types made with the `interface` and `type` keyword.

It is useful to think of these types as sets. In other words a type itself is a collection of possible things which your variables/properties can be at that time. For example:

```javascript
// the variable five can be one thing which is a number
const foo = 5; // type = number; 

// bar can be two things
type twoPrimitives = string | number;
let bar: twoPrimitives = 4; 
```

- We see foo has a set size of 1 >> {number}
- We see bar has a set size of 2 >> {number, string}

Now with our set definition in mind we can define the `never` type easy:

**the `never` type represents the empty type set >> {}**. i.e. a set with 0 items in it.

For example:

```javascript
// Normally a function which do not have a return value returns undefined implicitly
// However this function throws before ever reaching the end. 
// Therefore the return type will be never
function foo() :never {
   throw new Error("Error");
}

// thisMakesNoSense will be type never, a value can never be number and string at the same time
type thisMakesNoSense = number & string; 
```

---

# 1️⃣ **Exhaustive checks (most important real use case)**

You have a union type:

```ts
type Shape = "circle" | "square" | "triangle";
```

You handle each case:

```ts
function getArea(shape: Shape) {
  switch (shape) {
    case "circle": return 1;
    case "square": return 2;
    case "triangle": return 3;
    default:
      const _exhaustive: never = shape; // ❗ if a case is missing → error
      return _exhaustive;
  }
}
```

### Why do this?

If someone later adds a new shape:

```ts
type Shape = "circle" | "square" | "triangle" | "hexagon";
```

TypeScript will error here:

```
Type '"hexagon"' is not assignable to type 'never'.
```

✔ This tells you:  
**“You forgot to handle the new case!”**

This is the #1 reason `never` exists.  
It improves code safety in big apps.

---

# 2️⃣ **Functions that never return**

Example: a function that **always throws**:

```ts
function fail(): never {
  throw new Error("error");
}
```

Or **infinite loops**:

```ts
function loopForever(): never {
  while (true) {}
}
```

### Why important?

Because TypeScript can understand:

- after a `throw` → no execution continues
    
- after an infinite loop → no value comes back
    

This helps with control–flow analysis.

---

# 3️⃣ **Impossible situations**

```ts
function check(value: string | number) {
  if (typeof value === "string") {
    // value: string
  } else if (typeof value === "number") {
    // value: number
  } else {
    // value: never   ❗ impossible
  }
}
```

TS assigns `never` here because this branch cannot happen.

### Why useful?

Helps catch logic errors in complex type narrowing.

---

# 🧠 Why not just remove `never` and use `void`?

Because:

- `void` means “no useful return value”, but the function **returns normally**.
    
- `never` means **it never returns at all**.
    

Example:

```ts
function f1(): void {
  return; // allowed
}

function f2(): never {
  return; // ❌ error: cannot return
}
```

TypeScript needs both to reason correctly about code flow.

---

# 🎯 Summary (easy to remember)

| Use Case                        | Why `never` matters                  |
| ------------------------------- | ------------------------------------ |
| **Exhaustive switch checks**    | Catch missing cases (most important) |
| **Functions that never return** | Better control-flow analysis         |
| **Impossible branches**         | TS warns about unreachable code      |

---

### References

https://stackoverflow.com/questions/49219531/why-was-the-never-type-introduced-in-typescript