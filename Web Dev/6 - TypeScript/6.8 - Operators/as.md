---
tags: 
 - typescript
 - operator
---

## 🔧 What Is `as` in TypeScript?

`as` is a **type assertion operator**. It tells the TypeScript compiler:

> “Trust me — I know the type better than you here.”

```ts
const value = document.getElementById("app") as HTMLDivElement;
```

⚠️ `as` **does not change the runtime value**.  
It only affects **type checking**.

---

## 🧠 What `as` Actually Does

- Narrows or widens a type **at compile time**
    
- Overrides TypeScript’s inferred type
    
- Skips certain safety checks
    

```ts
const input = "123" as unknown as number;
```

TypeScript accepts this, even though it is unsafe at runtime.

---

## 🧩 Common Use Cases

### 📌 DOM Access

```ts
const btn = document.querySelector("button") as HTMLButtonElement;
```

Without `as`, TypeScript infers `Element | null`.

---

### 📦 Working with `unknown`

```ts
function parse(value: unknown) {
  return value as string;
}
```

`unknown` requires assertion before use.

---

### ⚛️ React `useRef`

```ts
const ref = useRef<HTMLDivElement>(null as HTMLDivElement | null);
```

Used when initialization cannot match final type.

---

## 🔄 `as` vs Angle-Bracket Syntax

```ts
// Preferred
const el = value as HTMLElement;

// Legacy / not allowed in JSX
const el = <HTMLElement>value;
```

📌 **`as` is required in JSX/TSX** because angle brackets conflict with JSX syntax.

---

## 🚨 Unsafe Assertions

```ts
const n = "hello" as number; // ❌ compiles, but wrong
```

`as` **does not validate** data.

Bad assertions:

- Hide bugs
    
- Break refactors
    
- Cause runtime crashes
    

---

## 🧪 Safe Narrowing vs Assertion

### Prefer narrowing:

```ts
if (typeof value === "string") {
  value.toUpperCase();
}
```

### Avoid unnecessary assertions:

```ts
(value as string).toUpperCase(); // weaker
```

---

## 🧊 `as const`

`as const` is a **special [[Type Assertion|assertion]]** that:

- Makes properties `readonly`
    
- Converts values to literal types
    

```ts
const config = {
  mode: "dark",
  retry: 3,
} as const;
```

Resulting type:

```ts
{
  readonly mode: "dark";
  readonly retry: 3;
}
```

`as const` can also be applied directly to a property, allowing TypeScript to infer that property as a **literal type** instead of widening it.

```ts
const buttonAttributes = {
  type: "button" as const, // in this case type of type property is "button" literal type
};

const buttonAttributes = {
  type: "button", // in this case type of type property is modified as readonly
} as const;
```


```ad-note
[[Object.freeze()]] run at runtime so its more costly and it does not go all the way down the nested object. `as const` recursively make the properties nested inside the object readonly.
```

```ad-warning
Rookie mistake: `as const` is not a type of [[Web Dev/6 - TypeScript/6.1 - Types/6.1.0 - Terms/Type Annotation|type annotation]].
```

---

## ⚠️ When NOT to Use `as`

🚫 To “fix” type errors without understanding them  
🚫 Instead of proper type guards  
🚫 For external data without validation

Use runtime validation for:

- API responses
    
- User input
    
- JSON parsing
    

---

## 🧾 Summary

- `as` is a **type assertion operator**
    
- It affects **only the TypeScript compiler**
    
- It performs **no runtime checks**
    
- Overuse reduces type safety
    
- `as const` is a common, safe pattern
    


---

## Related

[[as const with function return variable]]