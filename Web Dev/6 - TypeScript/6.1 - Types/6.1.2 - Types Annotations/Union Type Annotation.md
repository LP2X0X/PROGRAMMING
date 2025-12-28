---
tags: 
 - typescript
 - union
 - type
 - annotation
---

### 🔹 Definition

A **union type annotation** explicitly declares that a variable, parameter, or property can hold **values of multiple types** — written with the `|` (pipe) operator.

```ts
let value: string | number;
```

This tells TypeScript:

> "`value` can be either a `string` **or** a `number`."

---

### 🔹 Syntax

```ts
let variableName: Type1 | Type2 | Type3;
```

````ad-note
The or | symbol can be at the beginning also:
```ts
let variableName: | Type1 | Type2 | Type3;
```
````

✅ Examples:

```ts
let id: number | string;
let flag: boolean | "pending" | "failed";
let result: string | null | undefined;
```

---

### 🔹 Annotating Variables

```ts
let response: string | null = null;

response = "Success"; // ✅
response = 404;       // ❌ number not assignable
```

Union annotations keep variables flexible **but type-safe**.

---

### 🔹 Annotating Function Parameters and Return Types

```ts
function printId(id: string | number): void {
  console.log("ID:", id);
}

function toStringValue(value: number | boolean): string {
  return value.toString();
}
```

Here, the function explicitly accepts multiple input types.

---

### 🔹 Annotating Object Properties

You can use union annotations **inside object types**:

```ts
type User = {
  id: number | string;
  status: "active" | "inactive";
};

const u: User = { id: "U123", status: "active" };
```

---

### 🔹 Annotating Arrays of Union Types

Be careful with parentheses!

|Syntax|Meaning|
|---|---|
|`string|number[]`|
|`(string|number)[]`|

Example:

```ts
let data: (string | number)[] = [1, "hello", 2];
```

---

### 🔹 Annotating Function Signatures (Overload Style)

Sometimes you can use **union annotations** to represent multiple call signatures:

```ts
function format(value: string | number): string {
  return typeof value === "string" ? value.toUpperCase() : value.toFixed(2);
}
```

TypeScript infers narrowing based on the argument type.

---

### 🔹 Combining Union with Type Aliases

To improve readability and reuse:

```ts
type ID = number | string;
type Status = "online" | "offline" | "away";

let userId: ID = 42;
let state: Status = "online";
```

---

### 🧠 Type Narrowing with Union Annotation

When using union types, TypeScript forces you to **narrow** the type before using members that don’t exist on all variants.

```ts
function process(input: string | number) {
  if (typeof input === "string") {
    console.log(input.toUpperCase()); // ✅ safe
  } else {
    console.log(input.toFixed(2)); // ✅ safe
  }
}
```

Without narrowing, TypeScript will report errors like:

> “Property 'toUpperCase' does not exist on type 'string | number'.”

---

### ⚠️ Common Pitfalls

|Pitfall|Description|Fix|
|---|---|---|
|Accessing members without narrowing|TS doesn’t know which type is active|Use `typeof`, `instanceof`, or `"prop" in obj`|
|Ambiguous unions|e.g. `string|number|
|Forgetting parentheses in arrays|`string|number[]`≠`(string|

---

### 💡 Tips

- Use **type aliases** to simplify repetitive unions.
    
- Combine with **literal types** for restricted values.
    
- Keep unions **small** for readability and effective narrowing.
    
- Always **narrow** before accessing type-specific methods.
    

---

### 🧩 Example Summary

```ts
type Input = string | number;
type Mode = "auto" | "manual";

function handleInput(data: Input, mode: Mode) {
  if (typeof data === "string") console.log(data.toUpperCase());
  else console.log(data.toFixed(2));
}

handleInput(3.14, "auto");
```

---

### ✅ Quick Recap

|Concept|Description|Example|
|---|---|---|
|**Definition**|Type that can be one of several options|`string \| number`|
|**Used for**|Parameters, properties, return types, arrays|`let x: A \| B`|
|**Needs narrowing?**|Yes, before type-specific access|`typeof x === "string"`|
|**Best with**|Literal types, type aliases, narrowing checks|`"up" \| "down"`|