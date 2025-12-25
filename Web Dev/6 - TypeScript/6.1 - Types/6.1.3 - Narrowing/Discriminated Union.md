---
tags: 
 - typescript
 - narrowing
---

A **discriminated union** is:

- A **union of object types**
    
- Each member has a **shared property** (the _discriminant_)
    
- The discriminant has a **literal type** that uniquely identifies each variant
    

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number };
```

🧠 `kind` is the **discriminant property**.

---

### Why it exists 🎯

- Enables **safe and automatic type narrowing**
    
- Avoids unsafe type assertions
    
- Models real-world “one of many variants” data
    

---

## How narrowing works 📉

### Example 🔎

```ts
function area(shape: Shape) {
  if (shape.kind === "circle") {
    // shape is { kind: "circle"; radius: number }
    return Math.PI * shape.radius ** 2;
  } else {
    // shape is { kind: "square"; size: number }
    return shape.size ** 2;
  }
}
```

✔ TypeScript narrows based on the literal value of `kind`.

---

### `switch` is the common pattern 🔁

```ts
function area(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
  }
}
```

---

## Requirements ✅

A discriminated union needs:

1. **Union type** (`|`)
    
2. **Common property name**
    
3. **Literal types** (`"circle"`, `"square"`, not `string`)
    
4. **Distinct values** per variant
    

---

## Why literal types matter ⚠️

```ts
// ❌ No narrowing
type Bad =
  | { kind: string; radius: number }
  | { kind: string; size: number };
```

🧠 TypeScript cannot distinguish variants if the discriminant is not literal.

---

## Exhaustiveness checking 🧪

### Using `never` for safety 🔒

```ts
function area(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

✔ Compiler error if a new variant is added and not handled.

---

## Common use cases 🧠

- Redux actions
    
- State machines
    
- API response variants
    
- UI component states
    
- Error/result modeling
    

```ts
type Result =
  | { status: "success"; data: string }
  | { status: "error"; message: string };
```

---

## Comparison with other narrowing 🆚

| Technique           | Best for                   |
| ------------------- | -------------------------- |
| Discriminated union | Structured variant objects |
| `instanceof`        | Class instances            |
| `typeof`            | Primitives                 |
| Type predicates     | Complex runtime logic      |

---

## Key advantages ⭐

- Compile-time safety
    
- Self-documenting types
    
- No runtime cost
    
- Scales well as unions grow
    

---

## Summary 📝

- Discriminated union = union + shared literal property
    
- Enables precise, automatic narrowing
    
- Ideal for “one-of-many” object models
    
- Works best with `switch` and exhaustiveness checks
    