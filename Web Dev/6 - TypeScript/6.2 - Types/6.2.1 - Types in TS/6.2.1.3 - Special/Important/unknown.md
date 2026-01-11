---
tags: 
 - typescript
 - type
 - object
---

In TypeScript, **`unknown`** is a **type-safe alternative to `any`**. It represents a value whose type is not known _yet_, but unlike `any`, it **forces you to check the type before using it**.

![[065-introduction-to-unknown.explainer.svg|center|700]]
 <figcaption align="center"><strong>Figure:</strong> Assignability chart where all types can be assign to unknown but not vice versa. </figcaption>

---

## What `unknown` means

> “I do not know what this value is, and you must prove its type before operating on it.”

```ts
let value: unknown;

value = 42;
value = "hello";
value = {};
```

All assignments are allowed.

```ad-note
unknown sits at the top of the types hierarchy in TypeScript.
```

---

## Key difference: `unknown` vs `any`

### `any` (unsafe)

```ts
let v: any;
v.toUpperCase();   // OK at compile time, may crash at runtime
```

### `unknown` (safe)

```ts
let v: unknown;
v.toUpperCase();   // Error: Object is of type 'unknown'
```

You **must narrow** first.

---

## How to use an `unknown` value

### 1. Type narrowing with `typeof`

```ts
function print(value: unknown) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  }
}
```

### 2. Using `instanceof`

```ts
function handle(err: unknown) {
  if (err instanceof Error) {
    console.log(err.message);
  }
}
```

### 3. User-defined type guards

```ts
function isUser(x: unknown): x is { id: number } {
  return typeof x === "object" && x !== null && "id" in x;
}

if (isUser(value)) {
  value.id;
}
```

---

## Assignability rules (important)

### You can assign **anything → unknown**

```ts
let u: unknown;

u = 1;
u = "a";
u = true;
```

### You can assign **unknown → only unknown or any**

```ts
let x: unknown;
let y: string;

y = x;      // Error
y = x as string; // OK (explicit assertion)
```

This is what enforces safety.

---

## Common use cases

### 1. Catching errors safely

```ts
try {
  risky();
} catch (e: unknown) {
  if (e instanceof Error) {
    console.error(e.message);
  }
}
```

### 2. Parsing external data (JSON, API, user input)

```ts
const data: unknown = JSON.parse(input);

// must validate before use
```

### 3. Generic constraints and APIs

```ts
function process<T>(value: T): unknown {
  return value;
}
```

---

## `unknown` vs `never` vs `any`

| Type      | Meaning                   |
| --------- | ------------------------- |
| `any`     | Trust me, skip checking   |
| `unknown` | I don’t know yet—prove it |
| `never`   | This cannot happen        |

---

## Mental model

- Use **`unknown`** at **boundaries** (I/O, external data)
    
- Use **narrowing** before business logic
    
- Avoid `any` unless absolutely necessary
    

---

### One-line summary

> **`unknown` is the safest “I don’t know” type in TypeScript—it blocks usage until you narrow it.**