---
tags: 
 - typescript
 - type
 - compound
---

## 🧩 1. What is an Array in TypeScript?

An array is a list of elements of the **same type**.

TypeScript adds type safety to arrays so you can’t push incompatible values.

```ts
let numbers: number[] = [1, 2, 3];
```

If you try:

```ts
numbers.push("hello"); // ❌ error
```

---

# 🧩 2. Ways to Declare Arrays

## ✔ Method 1: Using `T[]` syntax (most common)

```ts
let names: string[] = ["a", "b", "c"];
```

## ✔ Method 2: Using `Array<T>`

```ts
let scores: Array<number> = [10, 20, 30];
```

They are identical — only style preference.

---

# 🧩 3. Arrays with Multiple Types (Union Arrays)

```ts
let mixed: (string | number)[] = [1, "hello", 5];
```

---

# 🧩 4. Readonly Arrays

Prevents mutation (`push`, `pop`, etc.).

```ts
const ids: readonly number[] = [1, 2, 3];

ids.push(4); // ❌ error
```

Or:

```ts
let data: ReadonlyArray<string> = ["a", "b"];
```

---

# 🧩 5. Type Inference Works Automatically

```ts
const items = [1, 2, 3];
// TS infers: number[]
```

But with mixed types:

```ts
const values = [1, "hello"];
// TS infers: (number | string)[]
```

---

# 🧩 6. Array Methods Are Type-Safe

### map

```ts
const lengths = ["a", "bbb"].map(s => s.length);
// inferred: number[]
```

### filter

```ts
const evens = [1, 2, 3, 4].filter(n => n % 2 === 0);
// inferred: number[]
```

### reduce

```ts
const sum = [1, 2, 3].reduce((acc, n) => acc + n, 0);
// inferred: number
```

---

# 🧩 7. Tuple vs Array (Important Distinction)

### ✔ Array = flexible, dynamic length

### ✔ Tuple = fixed length, fixed types per index

Tuple example:

```ts
const person: [string, number] = ["Long", 25];
```

Invalid:

```ts
person[0] = 100; // ❌ not allowed
```

---

# 🧩 8. Optional Array Elements

```ts
let arr: (number | undefined)[] = [1, 2, undefined];
```

---

# 🧩 9. Nested Arrays (2D arrays)

```ts
let grid: number[][] = [
  [1, 2],
  [3, 4],
];
```

---

# 🧩 10. Arrays in Interfaces and Types

```ts
interface User {
  name: string;
  hobbies: string[];
}
```

```ts
type Point = {
  position: [number, number];
};
```

---

# 🧩 11. Common Real-World Use Cases

### ✔ list of API data

```ts
interface Todo {
  id: number;
  title: string;
}

const todos: Todo[] = [];
```

### ✔ React state arrays

```ts
const [items, setItems] = useState<string[]>([]);
```

### ✔ filtering and mapping data

```ts
const titles = todos.map(t => t.title);
```

### ✔ storing coordinates, rows, grids

```ts
const matrix: number[][] = [];
```

### ✔ defining tuple responses (e.g., React hooks)

```ts
type Response = [data: string, error: string | null];
```

---

# 🧩 12. Best Practices

- Prefer `string[]` over `Array<string>` for simplicity
    
- Use `readonly string[]` when mutation should be prevented
    
- Use tuples for fixed structures (e.g., `[number, string]`)
    
- Use unions for mixed arrays `(string | number)[]`
    
- Always define type for arrays returned from APIs
    

---

# 📌 Summary (Quick Notes)

|Concept|Example|
|---|---|
|Standard array|`number[]`|
|Generic array|`Array<number>`|
|Union array|`(string|
|Readonly|`readonly number[]`|
|Nested array|`number[][]`|
|Tuple (fixed)|`[string, number]`|