---
tags: 
 - typescript
 - note
 - typeof
 - operator
---

## The Fundamental Rule

TypeScript makes a strict distinction between:

- **value-level expressions** (runtime JavaScript)
    
- **control-flow type guards** (compile-time narrowing)
    

`typeof` only becomes _special_ in the **second case**.

---

## Why `typeof` Is “Just JavaScript” Here

In your code:

```ts
const format = (input: string | number | boolean) => {
	const inputType = typeof input;
```

### What this means to TypeScript

1. `typeof input` is evaluated at **runtime**
    
2. The result is a **string**
    
3. That string is used as a **dynamic property key**
    
4. There is **no control-flow condition** involved
    

So TypeScript says:

> “I must assume this behaves exactly like JavaScript.”

And in JavaScript:

```js
typeof anything
```

can return **any of these**:

```ts
"string" | "number" | "bigint" | "boolean" |
"symbol" | "undefined" | "object" | "function"
```

TypeScript **cannot safely reduce that set** here.

---

## Why TypeScript Refuses to Be Smarter

Even though _you_ know:

```ts
input: string | number | boolean
```

TypeScript refuses to infer:

```ts
typeof input → "string" | "number" | "boolean"
```

because doing so would break **soundness**.

### Example of Unsoundness

```ts
let input: string | number | boolean = "hi";

const t = typeof input;

// JS allows this
input = { x: 1 } as any;

objOfFunctions[t]; // ❌ t no longer matches input
```

TypeScript must assume **mutation and aliasing are possible** between reads.

---

## Why `typeof` Works Differently in `if` Statements

Now compare:

```ts
if (typeof input === "string") {
  input.toUpperCase();
}
```

Here:

- `typeof input === "string"` is a **recognized narrowing pattern**
    
- TypeScript applies **control-flow analysis**
    
- Narrowing is **temporary and scoped**
    
- The value cannot change _inside the branch_
    

This is a **compiler rule**, not a runtime feature.

---

## Key Design Principle at Work

> **TypeScript narrows only when it can prove safety via control flow.**

Your code has:

- No branch
    
- No guard
    
- No invariant
    
- No guarantee the value stays the same
    

So TypeScript defaults to **pure JavaScript semantics**.

---

## Why This Is Actually a Good Thing

If TypeScript treated every `typeof` as a type guard:

- Indexing objects would become unsound
    
- Reassignment bugs would compile
    
- Type safety would silently degrade
    

So the rule is conservative **by design**.

---

## Mental Model to Remember

Think of it this way:

- `typeof x` → **runtime string**
    
- `typeof x === "string"` → **compile-time guarantee**
    
- Only **comparisons** trigger narrowing
    
- Everything else is treated as **ordinary JavaScript**
    

---

## One-Sentence Summary

> TypeScript treats `typeof` like normal JavaScript unless it appears in a recognized **control-flow narrowing pattern**, because only then can it guarantee soundness.