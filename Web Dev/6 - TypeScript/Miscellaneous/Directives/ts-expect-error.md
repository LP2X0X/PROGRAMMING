---
tags: 
 - typeScript
 - directive
---

`@ts-expect-error` is a **TypeScript comment directive (pragma)** that asserts the **next line must produce a type error**.

If the next line **does not** produce an error, TypeScript will report a compiler error.

---

### Syntax

```ts
// @ts-expect-error
<statement>
```

Must appear **immediately above** the line it applies to.

---

### Behavior

- If the next line **has a type error** → ✔ compilation continues
    
- If the next line **has no error** → ❌ compiler error: _Unused '@ts-expect-error' directive_
    

This makes it a **verified assertion**, not a suppression.

---

### Why it exists

- Documents **intentional type violations**
    
- Prevents silent regressions when types change
    
- Acts as a **type-level test**
    

---

### Common use cases

- Testing type definitions
    
- Working around known incorrect third-party typings
    
- Ensuring unsafe code paths stay invalid
    
- Migrating legacy code safely
    

```ts
// @ts-expect-error — library typing is incorrect
unsafeLibraryCall("valid at runtime");
```

---

### Compared to `@ts-ignore`

| Directive          | Behavior          | Safety |
| ------------------ | ----------------- | ------ |
| `@ts-expect-error` | Requires an error | High   |
| `@ts-ignore`       | Silences error    | Low    |

> Prefer `@ts-expect-error` whenever possible.

---

### Limitations

- Applies to **only the next line**
    
- Does not affect runtime
    
- Does not suppress non-type errors (e.g. syntax errors)
    

---

### Mental model

> **“This line should fail type-checking, and I want the compiler to enforce that expectation.”**

---

### Related directives

- `@ts-ignore`
    
- `@ts-nocheck`
    
- `@ts-check`
    
