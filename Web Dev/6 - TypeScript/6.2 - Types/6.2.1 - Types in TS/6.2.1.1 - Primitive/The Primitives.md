---
tags: 
 - typescript
 - type
 - primitives
---

## 🧩 1. What are primitives?

**Primitive types** are the simplest, immutable data types in TypeScript (and JavaScript).  
They are **not objects** — they have no methods or properties by themselves (though JS wraps them temporarily in objects when needed).

---

## 🧱 2. List of primitive types

| Type        | Example             | Notes                            |
| ----------- | ------------------- | -------------------------------- |
| `string`    | `"Hello"`           | Text data                        |
| `number`    | `42`, `3.14`, `NaN` | All numbers are floating-point   |
| `boolean`   | `true` / `false`    | Logical values                   |
| `bigint`    | `123n`              | For very large integers          |
| `symbol`    | `Symbol('id')`      | Unique identifiers               |
| `null`      | `null`              | Intentional absence of any value |
| `undefined` | `undefined`         | Uninitialized or missing value   |

> ⚠️ Primitives are **passed by value**, not by reference.

---

## 🧠 3. Common pitfalls & details

### ⚠️ a. **`null` vs `undefined`**

|Concept|Meaning|
|---|---|
|`undefined`|A variable has been declared but not assigned a value|
|`null`|Explicitly means “no value” or “empty”|

```ts
let a: undefined = undefined;
let b: null = null;
```

🪤 **Pitfall:**  
They’re often confused or used interchangeably — but TypeScript treats them differently when `--strictNullChecks` is on.

✅ **Tip:**  
Use **`undefined`** for “not assigned” (default case).  
Use **`null`** only when you intentionally clear a value.

---

### ⚠️ b. **Wrapper objects vs primitives**

Don’t use `new String()`, `new Number()`, `new Boolean()`!

```ts
let a = new String("hi");   // 🚫 Object, not string
typeof a;                   // "object"

let b: string = "hi";       // ✅ Primitive
```

🪤 **Pitfall:**  
`new String()` creates an object, which can lead to type mismatches or truthy/falsey confusion.

✅ **Tip:**  
Always use lowercase `string`, `number`, `boolean` for type annotations, not `String`, `Number`, `Boolean`.

---

### ⚠️ c. **NaN quirks**

`NaN` (Not-a-Number) is still of type `number`.

```ts
typeof NaN; // "number"
NaN === NaN; // false
```

🪤 **Pitfall:**  
NaN is not equal to itself.  
Use `Number.isNaN(value)` to check properly.

✅ **Tip:**  
Validate inputs before numeric operations.

---

### ⚠️ d. **BigInt**

Use `bigint` when numbers exceed JS’s safe integer limit (`2^53 - 1`).

```ts
let big: bigint = 9007199254740993n;
```

🪤 **Pitfall:**  
You **cannot mix** `number` and `bigint` in arithmetic:

```ts
const result = 10n + 5; // ❌ TypeError
```

✅ **Tip:**  
Stick to one numeric type for each context.

---

### ⚠️ e. **Symbol uniqueness**

Each symbol is unique, even with the same description:

```ts
Symbol("id") === Symbol("id"); // false
```

✅ **Tip:**  
Use symbols for keys that must not clash with others (e.g., internal object IDs).

---

### ⚠️ f. **Literal types**

You can constrain a variable to specific literal values:

```ts
let direction: "left" | "right" = "left";
direction = "up"; // ❌ Error
```

✅ **Tip:**  
Use literal types for safer enums and configuration constants.

---

## 🧩 4. TypeScript-specific notes

### 🔸 Type inference

TypeScript infers types automatically for primitives:

```ts
let message = "hello"; // inferred as string
```

✅ **Tip:**  
Let TS infer types for simple values — only annotate when needed.

---

### 🔸 Type widening & narrowing

TS “widens” a literal type to its base primitive unless you use `const`:

```ts
let x = "hi";     // type: string
const y = "hi";   // type: "hi"
```

✅ **Tip:**  
Use `const` when you want literal preservation (e.g., for discriminated unions).

---

### 🔸 `any`, `unknown`, and `never`

Although not true primitives, they relate to primitive safety:

|Type|Meaning|Example|
|---|---|---|
|`any`|Disables type checking|`let a: any = 5;`|
|`unknown`|Must be type-checked before use|`if (typeof x === "string") { ... }`|
|`never`|Value that never occurs|`function fail(): never { throw new Error() }`|

✅ **Tip:**  
Avoid `any` — prefer `unknown` or explicit unions.

---

## 💡 5. Quick summary cheat table

|Type|Example|Common Pitfall|Tip|
|---|---|---|---|
|`string`|`"hi"`|Using `String` instead|Always lowercase `string`|
|`number`|`42`|Mixing with `bigint`|Use `Number.isNaN()`|
|`boolean`|`true`|Using `new Boolean()`|Avoid wrapper objects|
|`bigint`|`10n`|Mixing with numbers|Stay consistent|
|`symbol`|`Symbol("id")`|Expecting equality|All symbols are unique|
|`null`|`null`|Misused for unassigned|Use only when deliberate|
|`undefined`|`undefined`|Forgetting strictNullChecks|Prefer default “missing”|

---

### 🧭 TL;DR

- ✅ Use **lowercase** primitive types (`string`, `number`, etc.)
    
- ⚠️ Avoid **wrapper objects** (`new String()`)
    
- 🔒 Enable **`strictNullChecks`** to avoid `null` bugs
    
- 🧠 Remember: primitives are **immutable** and **passed by value**
    
- 🚀 Use **literal types** for safety and clarity
    