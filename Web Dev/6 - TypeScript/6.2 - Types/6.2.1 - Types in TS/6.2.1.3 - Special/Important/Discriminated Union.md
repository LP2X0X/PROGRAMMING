---
tags: 
 - typescript
 - type
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

🧠 `kind` is the **discriminant property** (or *discriminator*).

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

```ts
function area(shape: Shape) {
  switch (true) {
    case shape.kind === "circle":
      return Math.PI * shape.radius ** 2;
    case shape.kind === "square":
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

## Default Discriminated Union

Imagine a UI Component that can either be a regular `Link` or a `Button`.

```ts
interface Button {
  type: 'button'; // The discriminator
  onClick: () => void;
}

interface Link {
  type?: "link"; // Effectively act as a "default" type
  href: string;
}

type ComponentProps = Button | Link;
```

When you check for the presence of the property, TypeScript will filter the possibilities.

```ts
function RenderComponent(props: ComponentProps) {
  if (props.type === 'button') {
    // Narrowed to Button
    return <button onClick={props.onClick}>Click Me</button>;
  } else {
	// Narrowed to Link (because if it's not a button, it's the other one)
	return <a href={props.href}>Go Here</a>;
  }
}
```

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
    