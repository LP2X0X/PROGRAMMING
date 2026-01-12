---
tags: 
 - typescript
 - type
 - special
---

![[Pasted image 20260111094921.png|center|500]]

---

## ⚠️ `{}` — The Most Misunderstood Type

```ts
let value: {};
```

### What `{}` Means

> Any value that is **not `null` or `undefined`**

```ts
value = 1;          // ✅
value = "hello";    // ✅
value = [];         // ✅
value = {};         // ✅
value = () => {};   // ✅

value = null;       // ❌
value = undefined;  // ❌
```

### Key Insight

- `{}` does **NOT** mean “object”
    
- It means **“non-nullish value”**
    

---

## 🔍 Why TypeScript Designed `{}` This Way

### Historical + Structural Typing Reasons

- TypeScript is **[[Structural Typing|structurally typed]]**
    
- `{}` has **no required properties**
    
- Therefore, any non-nullish value structurally satisfies it
    

This is a **type-system decision**, not a JS runtime rule.

---

## 🚫 Why `{}` Is Dangerous

```ts
function fn(x: {}) {
  x.toString(); // allowed
}
```

You may think `x` is an object — it might be a number.

---

## 🧱 `object` — Any Non-Primitive Object

```ts
let value: object;
```

### What `object` Means

> Any value that is **not a primitive**

❌ primitives:

- string
    
- number
    
- boolean
    
- symbol
    
- bigint
    
- null
    
- undefined
    

✅ allowed:

```ts
value = {};
value = [];
value = () => {};
value = new Date();
```

---

## 🧠 Use `object` When

- You need “some object”
    
- You do not care about its properties
    
- You want to exclude primitives
    

---

## 🟩 `Object` — Almost Never What You Want

```ts
let value: Object;
```

### What `Object` Means

- Global JavaScript `Object` type
    
- Extremely permissive
    
- Mostly legacy
    

Allows primitives via boxing:

```ts
value = 1; // ✅
```

🚫 **Avoid in modern TypeScript**

---

## ❌ Why `Object` Is Discouraged

- Weak typing
    
- Legacy semantics
    
- Confusing behavior
    
- Rarely useful
    

---

## 🧩 `Record<PropertyKey, never>` — A Truly Empty Object

```ts
type Empty = Record<PropertyKey, never>;
```

### What It Means

> An object with **no allowed properties**

```ts
const a: Empty = {};        // ✅
const b: Empty = { x: 1 }; // ❌
```

This is the **only true “empty object”**.

---

## 🧠 Best Use Case

- Enforcing “no props”
    
- Marker objects
    
- Strict APIs
    

---

## 🧪 `{}` vs `Record<string, never>`

```ts
function f1(x: {}) {}
function f2(x: Record<string, never>) {}
```

| Input      | f1  | f2  |
| ---------- | --- | --- |
| `{}`       | ✅  | ✅  |
| `{ a: 1 }` | ✅  | ❌  |
| `123`      | ✅  | ❌  |

---

## 🧠 Empty Object as Default Generic

❌ Bad:

```ts
function fn<T = {}>() {}
```

`T = {}` allows **almost anything**.

✅ Better:

```ts
function fn<T = unknown>() {}
```

---

## ⚛️ React Props Pitfall

```ts
type Props = {};
```

This allows:

```ts
<Component anything={123} /> // ❌ unexpected
```

✅ Correct:

```ts
type Props = Record<string, never>;
```

---

## 🧠 Mental Model

|Type|Meaning|
|---|---|
|`{}`|Non-nullish value|
|`object`|Non-primitive|
|`Object`|Legacy global|
|`Record<string, never>`|Truly empty object|

---

## 📌 Best Practices Summary

- 🚫 Avoid `{}` as an object type
    
- ✅ Use `object` for unknown objects
    
- ✅ Use `Record<string, never>` for empty objects
    
- 🚫 Avoid `Object`
    
- 🧠 Prefer `unknown` for safety
    

---

## 🧠 One-Sentence Summary

> In TypeScript, `{}` does **not** mean “empty object”; `Record<string, never>` does — and confusing these leads to subtle, dangerous bugs.

---

## References

https://www.totaltypescript.com/the-empty-object-type-in-typescript
https://github.com/sindresorhus/type-fest/issues/395

---

# Advance Stuff

https://github.com/sindresorhus/type-fest/issues/395

This is a sophisticated **TypeScript hack** used to define a strictly **Empty Object**.

It solves a specific problem where standard solutions (like `Record<string, never>` or `{}`) fail to correctly block objects with properties, or fail to work with interfaces.

Here is the breakdown of why this code exists and how it works.

---

### 1. The Goal

We want a type `EmptyObject` that:

1. **Accepts** `{}` (a truly empty object).
    
2. **Rejects** `{ name: "John" }` (objects with extra properties).
    
3. **Works** with Interfaces (which standard solutions often fail to do).
    

### 2. The Mechanics: "Weak Type" Safety

This solution leverages a TypeScript feature introduced in version 2.4 called **Weak Type Detection**.

A Weak Type is any type that contains only optional properties.

TypeScript enforces a strict rule for these:

> _"If you assign a value to a Weak Type, the value **must** share at least one property with that type (or be completely empty)."_

Let's trace the logic of your snippet:

#### Step A: The Unique Symbol

```ts
declare const tag: unique symbol;
```

This creates a property key that is impossible for anyone else to accidentally use. Since it is not exported, no object you create manually (like `{ name: 'foo' }`) can possibly contain this specific `tag` symbol as a key.

#### Step B: The Definition

```ts
type EmptyObject = { [tag]?: never };
```

- **`[tag]?`**: Makes the property optional. Since this is the _only_ property, `EmptyObject` becomes a **Weak Type**.
    
- **`never`**: Just in case someone _did_ have access to the symbol, this ensures they can't actually assign a value to it.
    

#### Step C: The Check (How TS thinks)

**Scenario 1: Assigning a populated object**

```ts
const user = { name: "Alice" };
const empty: EmptyObject = user; // ❌ ERROR
```

- **TS Logic:** `EmptyObject` is a Weak Type. Does `user` share any properties with it?
    
- **Check:** Does `user` have the property `[tag]`? -> **No.**
    
- **Result:** Error. Type `{ name: string }` has no properties in common with `EmptyObject`.
    

**Scenario 2: Assigning an empty object**

```ts
const nothing = {};
const empty: EmptyObject = nothing; // ✅ OK
```

- **TS Logic:** TypeScript makes a specific exception for the empty object literal `{}` to allow it to initialize Weak Types.
    

---

### 3. Why not use standard approaches?

You might ask, "Why not just use these simpler types?"

#### Option A: `type Empty = {}`

This is dangerously misleading. In TypeScript, `{}` actually means "Any non-null value".

```ts
const x: {} = "hello"; // ✅ This is allowed! (Bad)
const y: {} = { a: 1 }; // ✅ This is allowed! (Bad)
```

#### Option B: `type Empty = Record<string, never>`

This is the most common alternative, but it breaks when using **Interfaces**.

```ts
interface User {} // Interfaces are not "indexable" by default
const u: User = {};

type StrictEmpty = Record<string, never>;
const x: StrictEmpty = u; 
// ❌ ERROR: Index signature for type 'string' is missing in type 'User'.
```

### Summary

The `unique symbol` + `Weak Type` pattern is the most robust way to create a "Black Hole" type that only accepts `{}` and rejects everything else, while remaining compatible with interfaces and classes.

**It forces TypeScript to check for "property overlap" instead of "structural compatibility," and since the symbol is unique, overlap is impossible for anything except the empty set.**