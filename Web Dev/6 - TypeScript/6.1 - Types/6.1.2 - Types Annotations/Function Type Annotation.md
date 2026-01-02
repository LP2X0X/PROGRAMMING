---
tags: 
 - typescript
 - function 
 - type
 - annotation
---

TypeScript allows you to specify the types of both the input and output values of functions.

---

### Parameter Type Annotations

- When you declare a function, you can add type annotations after each parameter to declare what types of parameters the function accepts. Parameter type annotations go after the parameter name:

```ts
// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

- When a parameter has a type annotation, arguments to that function will be checked:

```ts
// Would be a runtime error if executed!
greet(42);
// Argument of type 'number' is not assignable to parameter of type 'string'.
```

```ad-note
Even if you don’t have type annotations on your parameters, TypeScript will still check that you passed the right number of arguments.
```

---

### Return Type Annotations

- You can also add return type annotations. Return type annotations appear after the parameter list:

```ts
function getFavoriteNumber(): number {
  return 26;
}
```

````ad-warning
This mean that function return type could be wider than what it returns:
```ts
const returnsStringOrNumber = (): string | number => {
  return 1;
};
```
````

- Much like variable type annotations, you usually don’t need a return type annotation because TypeScript will infer the function’s return type based on its return statements.

````ad-note
Function return types can help enforce the type of the function:
```ts
type UserRole = "admin" | "editor" | "viewer";

function getPermissions(role: UserRole): string[] {
  switch (role) {
    case "admin":
      return ["create", "read", "update", "delete"];
    case "editor":
      return ["create", "read", "update"];
    // case "viewer":
    //   return ["read"];
  }
}
```
````

Arrow function can also have annotated return type:
```ts
import { Equal, Expect } from "@total-typescript/helpers";

const fetchData = async (): Promise<[Error | undefined, any?]> => {
  const result = await fetch("/");

  if (!result.ok) {
    return [new Error("Could not fetch data.")];
  }

  const data = await result.json();

  return [undefined, data];
};

const example = async () => {
  const [error, data] = await fetchData();

  type Tests = [
    Expect<Equal<typeof error, Error | undefined>>,
    Expect<Equal<typeof data, any>>
  ];
};
```

---

### Functions Which Return Promises

- If you want to annotate the return type of a function which returns a promise, you should use the Promise type:

```ts
async function getFavoriteNumber(): Promise<number> {
  return 26;
}
```

---

### Anonymous Functions

Anonymous functions are a little bit different from function declarations. When a function appears in a place where TypeScript can determine how it’s going to be called, the parameters of that function are automatically given types.

```ts
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
  console.log(s.toUpperCase());
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

Even though the parameters didn’t have a type annotation, TypeScript used the types of the forEach function, along with the inferred type of the array, to determine the type s will have.

This process is called **[[Contextual Typing|contextual typing]]** because the context that the function occurred within informs what type it should have.

Similar to the inference rules, you don’t need to explicitly learn how this happens, but understanding that it does happen can help you notice when type annotations aren’t needed. 

---

## When to annotate a return type?

In TypeScript, deciding whether to annotate a return type or let the compiler "infer" it is a balance between **strictness** and **convenience**.

Here is the general rule of thumb used by most professional engineering teams:

### 1. When you SHOULD annotate

You should explicitly write the return type when the function is a "contract" or part of your public API.

- **Exported Functions/APIs:** If other files use this function, an explicit return type acts as a contract. If you accidentally change the logic, TypeScript will catch the mistake _inside_ the function, rather than at the place where the function is called.
    
- **Complex Logic:** In long functions with multiple `if/else` or `switch` statements, it’s easy to accidentally return the wrong shape in one branch. Annotating ensures all paths return what you expect.
    
- **Recursive Functions:** TypeScript often struggles to infer types for functions that call themselves; an annotation is usually required here.
    
- **Preventing `any` Leakage:** If your function calls a third-party library that returns `any`, annotating your function ensures that the `any` doesn't "infect" the rest of your codebase.
    

**Example of caught error:**

```ts
// If I forget the return type, TS might infer this as string | number by mistake
const getPrice = (discount: boolean): number => {
  if (discount) return 10;
  return "20"; // ❌ TS Error here because I annotated ': number'
};
```

### 2. When you SHOULD NOT annotate

You should let TypeScript infer the type when it’s redundant or makes the code harder to read without adding value.

- **Simple One-liners:** If the function is a simple transformation, the annotation just adds "noise."
    
- **Array Callbacks:** When using `.map`, `.filter`, or `.reduce`, the return type is usually very obvious from the context.
    
- **Local/Private Helpers:** If a function is only used a few lines away within the same file, inference is usually sufficient.
    

**Example of redundant code:**

```ts
// ❌ Over-annotated (Noise)
const add = (a: number, b: number): number => a + b;

// ✅ Clean (Inference)
const add = (a: number, b: number) => a + b; 
```

### Comparison: Inference vs. Annotation

| **Feature**     | **Inference (No Annotation)**                  | **Explicit Annotation**                               |
| --------------- | ---------------------------------------------- | ----------------------------------------------------- |
| **Speed**       | Faster to write.                               | Slower to write.                                      |
| --------------- | ---------------------------------------------- | ----------------------------------------------------- |
| **Readability** | Clean for simple logic.                        | Clear for complex logic.                              |
| **Refactoring** | Changing logic updates the type automatically. | Changing logic forces you to update the type (safer). |
| **Debugging**   | Errors appear where function is **called**.    | Errors appear where function is **defined**.          |

### The "Pro" Strategy: Test-Driven Types

A great middle-ground is to **annotate your interfaces and types first**, and then let the implementation be inferred.

For example, if you define a specific Type for a function, you don't need to annotate the return type because it's already part of the variable definition:

```ts
type SearchFn = (query: string) => string[];

// The return type is already known to be string[] because of 'SearchFn'
const findUsers: SearchFn = (query) => {
  return ["Alice", "Bob"].filter(name => name.includes(query));
};
```

### Summary Checklist

1. **Is it exported?** → Annotate.
    
2. **Is it a complex `if/else`?** → Annotate.
    
3. **Is it a simple helper or callback?** → Infer.
    
4. **Is it a recursive function?** → Annotate.
    