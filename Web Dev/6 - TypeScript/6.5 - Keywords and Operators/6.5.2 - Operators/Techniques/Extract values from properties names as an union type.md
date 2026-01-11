---
tags: 
 - typescript
 - operator
 - note
---

## Prerequisite
[[Indexed Access Type]]

```ad-warning
We can (should) never do it on a normal object though. It go against what TypeScript is trying to do which is not interfere with what's going on at runtime.
```

---

```ts
const Colors = { 
	Red: "#FF0000",
	Green: "#00FF00", 
	Blue: "#0000FF" 
} as const; // Makes properties readonly and literal types

type Color = typeof Colors[keyof typeof Colors];
```

This is **type-level property access** applied to a value.

---

## Step 1: `typeof Colors`

In a **type position**, `typeof` converts a **value** into its **type**.

```ts
typeof Colors
```

Becomes:

```ts
{
  readonly Red: "#FF0000";
  readonly Green: "#00FF00";
  readonly Blue: "#0000FF";
}
```

Why?

- `as const` makes:
    
    - properties `readonly`
        
    - values **string literals**, not `string`
        

---

## Step 2: `keyof typeof Colors`

`keyof` extracts the **property names** of that object type.

```ts
keyof typeof Colors
```

Becomes:

```ts
"Red" | "Green" | "Blue"
```

This is a **union of keys**.

---

## Step 3: Indexing with a Union of Keys

Now this part:

```ts
typeof Colors[keyof typeof Colors]
```

Means:

> “Give me the type of the property values for **all** keys of `Colors`.”

Expanded form:

```ts
typeof Colors["Red" | "Green" | "Blue"]
```

Which TypeScript evaluates as:

```ts
typeof Colors["Red"]
| typeof Colors["Green"]
| typeof Colors["Blue"]
```

---

## Step 4: Final Result

Each indexed access resolves to a literal type:

```ts
"#FF0000" | "#00FF00" | "#0000FF"
```

So:

```ts
type Color = "#FF0000" | "#00FF00" | "#0000FF";
```

---

## Key Mental Model

### Indexing a type with a union:

```ts
T[A | B | C]  ≡  T[A] | T[B] | T[C]
```

This rule is what makes the pattern work.

---

## Why This Is Powerful

- Keeps **runtime values and types in sync**
    
- Prevents duplication
    
- Replaces many enum use cases
    
- Fully erasable (works with `strip-types`)
    

---

## One-Line Intuition

> **“Take the object’s type, take all its keys, then collect the types of their values into a union.”**