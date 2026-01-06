---
tags: 
 - typescript
 - type
 - technique
---

When you use `as const` on the returned value of a function without explicitly defining the function's return type, TypeScript **infers the most specific (narrowest) possible type** for that return value.

Here is the breakdown of exactly what happens to the inferred type.

### 1. Types are "Narrowed" to Literals

Normally, TypeScript "widens" values to their general types (e.g., `"hello"` becomes `string`). With `as const`, TypeScript keeps them as **literal types**.

- **Without `as const`:** Returns `string`, `number`, `boolean`.
    
- **With `as const`:** Returns `"specific_string"`, `100`, `true`.
    

### 2. Objects Become `readonly`

All properties inside a returned object become `readonly`. TypeScript assumes this object will not change, so it locks the types down to the exact values provided.

### 3. Arrays Become `readonly` Tuples

Standard arrays (e.g., `string[]`) are mutable (you can push/pop). With `as const`, arrays become **read-only tuples** with a fixed length and specific order.

---

### Visual Comparison

Let's look at a concrete example to see the difference in the inferred return type.

#### Scenario A: Without `as const` (Standard Inference)

TypeScript assumes you might want to change these values later, so it infers general types.

```ts
function getSettings() {
  return {
    theme: "dark",
    retries: 3,
    supported: [100, 200]
  };
}

const config = getSettings();

// ❌ The Inferred Return Type is mutable and general:
// {
//   theme: string;         // Widened from "dark" to string
//   retries: number;       // Widened from 3 to number
//   supported: number[];   // Mutable array
// }

config.theme = "light"; // ✅ Allowed
config.supported.push(300); // ✅ Allowed
```

#### Scenario B: With `as const` (Const Assertion)

TypeScript freezes the inference to exactly what is written.

```ts
function getSettings() {
  return {
    theme: "dark",
    retries: 3,
    supported: [100, 200]
  } as const; // <--- The Magic
}

const config = getSettings();

// ✅ The Inferred Return Type is strict and readonly:
// {
//   readonly theme: "dark";            // Literal type
//   readonly retries: 3;               // Literal type
//   readonly supported: readonly [100, 200]; // Readonly Tuple
// }

config.theme = "light"; // ❌ Error: Cannot assign to 'theme' because it is a read-only property.
config.supported.push(300); // ❌ Error: Property 'push' does not exist on type 'readonly [100, 200]'.
```

### When is this useful?

1. **Custom Hooks:** When returning an array from a React hook (like `useState`), using `as const` ensures TypeScript treats it as a tuple `[value, setter]` rather than a union array `(value | setter)[]`.
    
2. **Redux/Action Creators:** It ensures the `type` property of an action is a specific string literal (e.g., `"LOGIN_SUCCESS"`) rather than just `string`, which is crucial for switch statements in reducers.
    
3. **Config Objects:** When defining configuration maps that shouldn't be mutated.
    

### Summary Table

| **Feature**     | **Without as const**     | **With as const**         |
| --------------- | ------------------------ | ------------------------- |
| **Primitives**  | `string`, `number`       | Literal (`"text"`, `1`)   |
| **Objects**     | Mutable properties       | `readonly` properties     |
| **Arrays**      | `T[]` (Mutable array)    | `readonly [T, U]` (Tuple) |
| **Flexibility** | High (can change values) | Low (immutable)           |

</br>

----

</br>

**`as const` is the canonical way to make a function call infer a tuple instead of a widened array in TypeScript.**

```ts
import { Equal, Expect } from "@total-typescript/helpers";

// Here, the return type of fetchData is Promise<readonly [Error] | readonly [undefined, any]]>
const fetchData = async () => {
  const result = await fetch("/");

  if (!result.ok) {
    return [new Error("Could not fetch data.")] as const;
  }

  const data = await result.json();

  return [undefined, data] as const;
};

const example = async () => {
  // First element is infer as either an Error or undefined 
  // Second element is infer as whatever JSON returns (any)
  const [error, data] = await fetchData();

  type Tests = [
    Expect<Equal<typeof error, Error | undefined>>,
    Expect<Equal<typeof data, any>>,
  ];
};
```

Below is the precise explanation.

---

## 🧠 The Problem: Array Widening

Without `as const`, array literals are **widened**.

```ts
function pair(values: readonly [string, number]) {}

pair(["id", 1]); 
// ❌ Error
// Type 'string[]' is not assignable to type 'readonly [string, number]'
```

TypeScript infers:

```ts
["id", 1] → (string | number)[]
```

---

## ✅ Solution: `as const`

```ts
pair(["id", 1] as const); // ✅
```

Now TypeScript infers:

```ts
readonly ["id", 1]
```

Which is compatible with:

```ts
readonly [string, number]
```

---

## 🔧 Why `as const` Works

`as const` does **three things at once**:

1. Prevents widening (`"id"` stays `"id"`)
    
2. Converts array literals to **tuples**
    
3. Makes the result `readonly`
    

```ts
const x = ["a", 1] as const;
// readonly ["a", 1]
```

---

## 🧩 Generic Function Example

```ts
function makeTuple<T extends readonly unknown[]>(value: T): T {
  return value;
}

const t1 = makeTuple(["a", 1]);        // (string | number)[]
const t2 = makeTuple(["a", 1] as const); // readonly ["a", 1]
```

Contextual typing + `as const` preserves tuple shape.

---

## 🧪 Alternative: Variadic Tuple Inference (When Possible)

Sometimes `as const` is unnecessary if the function signature captures tuples:

```ts
function tuple<T extends readonly unknown[]>(...args: T): T {
  return args;
}

const t = tuple("a", 1);
// inferred as ["a", 1]
```

But for **array literals passed as a single argument**, `as const` is required.

---

## ⚠️ Important Limitations

- `as const` makes the tuple **readonly**
    
- You cannot mutate it afterward
    

```ts
t2.push(3); // ❌ Error
```

If mutability is required, you must copy.

---

## 🧾 Summary

- Array literals widen to `T[]` by default
    
- `as const` forces tuple inference
    
- It preserves literal types and order
    
- It produces `readonly` tuples
    
- Essential for precise tuple-based APIs
    