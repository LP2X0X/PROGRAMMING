---
tags: 
 - typescript
 - operator
---

## 🧩 1. What is the Rest Operator?

The **rest operator** (`...`) collects multiple elements into **one variable**.

It means:

> “Take the _rest_ of the values and put them inside an array (or object).”

It works in:

- **Function parameters**
    
- **Array destructuring**
    
- **Object destructuring**
    

---

# 🧩 2. Rest Operator in Function Parameters

(Most common use)

### Syntax:

```ts
function fn(...args: Type[]) {}
```

### Example:

```ts
function sum(...nums: number[]) {
  return nums.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3); // 6
```

- `...nums` gathers all passed arguments into an array
    
- `nums` becomes a `number[]`
    

---

# 🧩 3. Rest in Array Destructuring

```ts
const [first, ...rest] = [10, 20, 30, 40];

first; // 10
rest;  // [20, 30, 40]
```

Collects all remaining items.

---

# 🧩 4. Rest in Object Destructuring

```ts
const user = { name: "Long", age: 25, country: "VN" };

const { name, ...info } = user;

name; // "Long"
info; // { age: 25, country: "VN" }
```

`info` gathers the rest of the key-value pairs into a new object.

---

# 🧩 5. Very Important: Rest Must Be the Last Item

### ❌ Wrong:

```ts
function bad(a, ...rest, c) {}
```

### ✔ Correct:

```ts
function good(a, ...rest) {}
```

Same rule for array & object destructuring:

```ts
const [...rest, last] = arr; // ❌ not allowed
```

---

# 🧩 6. Rest Operator vs Spread Operator

They look the same (`...`) but do opposite things.

### ✔ Rest = **collect**

```ts
function fn(...args) {} // collect
```

### ✔ Spread = **expand**

```ts
const arr2 = [...arr]; // expand
```

More examples:

**Rest (collect into array):**

```ts
const [a, ...rest] = [1, 2, 3];
```

**Spread (expand array):**

```ts
const arr = [1, 2, 3];
const newArr = [...arr, 4];
```

---

# 🧩 7. TypeScript and Rest Operator

### Function rest params

```ts
function log(...messages: string[]) {}
```

### Union types

```ts
function mix(...items: (string | number)[]) {}
```

### Tuple types

```ts
function config(...args: [string, number]) {}
```

### Variadic tuple types

```ts
function identity<T extends any[]>(...args: [...T]): T {
  return args;
}
```

---

# 🧩 8. Real-World Use Cases

### ✔ Logger functions

```ts
function log(...messages: string[]) {
  console.log(messages.join(" "));
}
```

### ✔ Forwarding all arguments

(Useful in decorators and wrappers)

```ts
function wrap(fn: (...args: any[]) => void) {
  return (...args: any[]) => fn(...args);
}
```

### ✔ Copying arrays with spread + rest

```ts
const arr = [1, 2, 3];
const [first, ...others] = arr;
```

### ✔ Removing the first element

```ts
const [, ...rest] = ["a", "b", "c"];
// rest: ["b", "c"]
```

### ✔ Object cloning minus a key

```ts
const { password, ...safeUser } = user;
```

---

# 📌 Summary (Quick Notes)

|Concept|Description|
|---|---|
|Rest Operator|collects remaining items|
|Syntax|`...name`|
|Works In|function params, array destructuring, object destructuring|
|Rule|must be last item|
|Returns|array or object|
|Opposite of|spread operator|
|TS Support|typed with array types: `...nums: number[]`|
