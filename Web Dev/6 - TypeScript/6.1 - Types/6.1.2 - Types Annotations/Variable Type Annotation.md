---
tags: 
 - typescript
 - type
 - annotation
 - variable
---

- When you declare a variable using `const`, `var`, or `let`, you can optionally add a type annotation to explicitly specify the type of the variable:

```ts
let myName: string = "Alice";
```

- In most cases, though, this isn’t needed. Wherever possible, TypeScript tries to automatically _infer_ the types in your code. For example, the type of a variable is inferred based on the type of its initializer:

```ts
// No type annotation needed -- 'myName' inferred as type 'string'  
let myName = "Alice";
```
