---
tags: 
 - typescript
 - function
 - rest
---

## 🧩 1. What is a Rest Parameter?

A **rest parameter** allows a function to take **any number of arguments** and stores them in an array.

Syntax:

```ts
function fn(...args: number[]) {}
```

### Meaning:

- `...` → gather remaining arguments
    
- `args` → name of the array
    
- `number[]` → type of all collected arguments
    

---

# 🧩 2. Basic Example

```ts
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3, 4); // 10
```

---

# 🧩 3. Rest Parameter Rules

### ✔ Must be the **last** parameter

```ts
function log(message: string, ...args: any[]) {}
```

### ✔ A function can have **only one** rest parameter

```ts
function bad(...a: number[], ...b: number[]) {} // ❌ not allowed
```

### ✔ Rest parameter’s type must be an array type

---

# 🧩 4. TypeScript + Rest = Strong Typing

## ✔ All arguments must match the rest type

```ts
function printNames(...names: string[]) {}

printNames("Alice", "Bob");   // ✔
printNames("Alice", 10);      // ❌ number not allowed
```

---

# 🧩 5. Using Union Types in Rest Parameters

```ts
function mix(...values: (string | number)[]) {}
```

---

# 🧩 6. Typing Tuples with Rest Parameters

TypeScript allows using **tuple types** as rest parameters.

### Example: Function expects `[number, number, string]`

```ts
function foo(...args: [number, number, string]) {
  // args: [number, number, string]
}

foo(1, 2, "hi");      // ✔
foo(1, "x", "hi");    // ❌
```

Great for cases where argument count is fixed but easier to write with rest syntax.

---

# 🧩 7. Advanced: Variadic Tuple Types

Introduced in TS 4.0 → rest parameters can mirror tuple types.

### Define a function that preserves argument types:

```ts
function identity<T extends any[]>(...args: [...T]): T {
  return args;
}

const result = identity(1, "hello", true);
// type: [number, string, boolean]
```

This is used a lot in **React** (e.g., typing custom hooks).

---

# 🧩 8. Rest Parameters + Destructuring

```ts
function list(first: string, ...rest: string[]) {
  console.log(first, rest);
}

list("a", "b", "c");
// first = "a"
// rest = ["b", "c"]
```

---

# 🧩 9. Using Rest in Higher-order Functions

### Example: decorator wrapper

```ts
function logCalls<F extends (...args: any[]) => any>(fn: F) {
  return (...args: Parameters<F>): ReturnType<F> => {
    console.log("Called with", args);
    return fn(...args);
  };
}
```

Here, rest parameters help forward arguments safely.

---

# 🧩 10. Real-world Use Cases

### ✔ Forwarding all arguments

(e.g., wrapper functions, decorators)

```ts
function call(fn: (...args: any[]) => void, ...args: any[]) {
  fn(...args);
}
```

### ✔ Event handlers (React)

```ts
function handle(...args: any[]) {}
```

### ✔ Logging utilities

```ts
function log(...messages: string[]) {
  console.log(messages.join(" "));
}
```

### ✔ Dynamic math operations

```ts
function multiply(...nums: number[]) {}
```

### ✔ Command-line argument parsing

```ts
parseArgs(...process.argv.slice(2));
```

### ✔ Strict tuple inputs

```ts
function useConfig(...args: [url: string, retry: number]) {}
```

---

# 🧩 11. Common Mistakes

### ❌ Using the wrong type (not an array)

```ts
function foo(...x: number) {} // ❌ must be number[]
```

### ❌ Placing more parameters after a rest

```ts
function bad(...a: string[], b: number) {} // ❌
```

### ❌ Assuming mixed types are allowed automatically

You must explicitly allow them:

```ts
function sample(...args: (string | number)[]) {} // ✔
```

---

# 📌 **Summary (Quick Notes)**

| Concept           | Explanation                                       |
| ----------------- | ------------------------------------------------- |
| Rest parameter    | Collects remaining arguments into an array        |
| Syntax            | `(...args: Type[])`                               |
| Must be last      | Yes                                               |
| Can only be one   | Yes                                               |
| Accepts unions    | Yes                                               |
| Works with tuples | Yes (`...args: [string, number]`)                 |
| Useful in         | Logging, wrappers, React handlers, spreading args |