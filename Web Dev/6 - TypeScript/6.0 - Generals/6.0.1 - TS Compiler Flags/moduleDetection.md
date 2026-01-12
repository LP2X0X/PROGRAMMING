---
tags: 
 - typescript
 - compiler
 - flag
---

In TypeScript, **`moduleDetection`** controls **how the compiler decides whether a file is a script or a module**. This matters because it affects **scoping, global pollution, and type checking behavior**.

---

## 1. What `moduleDetection` Does

Normally, TypeScript uses a **syntactic rule**:

> A file is a **module** if it contains **any `import` or `export`**.

`moduleDetection` lets you **override or refine this behavior**, especially for mixed or legacy codebases.

---

## 2. Available `moduleDetection` Modes

```json
{
  "compilerOptions": {
    "moduleDetection": "auto"
  }
}
```

### 2.1 `auto` (default)

TypeScript decides based on:

- Presence of `import` / `export`
    
- File extension (`.mts`, `.cts` are always modules)
    
- `package.json` `"type": "module"` (for `.ts` → `.js` emit)
    

**Behavior**

- Backward-compatible
    
- Best choice for most projects
    

---

### 2.2 `force`

**Every file is treated as a module**, even if it has no `import` or `export`.

```json
{
  "compilerOptions": {
    "moduleDetection": "force"
  }
}
```

**Why this exists**

- Prevents accidental globals
    
- Ideal for large or strict codebases
    
- Eliminates script-style cross-file globals
    

```ts
// fileA.ts
let x = 1;

// fileB.ts
x; // ❌ Error under `force`
```

---

### 2.3 `legacy`

Uses **old TypeScript behavior** (pre-4.7 style).

```json
{
  "compilerOptions": {
    "moduleDetection": "legacy"
  }
}
```

**Behavior**

- `.ts` files without imports/exports are scripts
    
- Ignores `.mts` / `.cts` semantics
    
- Not recommended for new projects
    

---

## 3. Interaction with File Extensions

| Extension | Module?                          | Notes            |
| --------- | -------------------------------- | ---------------- |
| `.ts`     | Depends on `moduleDetection`     | Default behavior |
| `.mts`    | Always module                    | ESM              |
| `.cts`    | Always module                    | CommonJS         |
| `.d.ts`   | Always module if `export` exists | Otherwise global |

---

## 4. Why `moduleDetection` Matters

### Preventing Accidental Globals

```ts
// utils.ts
function helper() {}
```

- `auto` → script (global)
    
- `force` → module (scoped)
    

---

### Enforcing Modern Architecture

With `force`, you **must explicitly import everything**, which:

- Improves readability
    
- Improves refactoring safety
    
- Matches runtime module behavior
    

---

## 5. Recommended Settings

### Modern TypeScript App

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleDetection": "force"
  }
}
```

### Legacy / Migration Project

```json
{
  "compilerOptions": {
    "moduleDetection": "auto"
  }
}
```

---

## 6. Key Takeaway (One Sentence)

> **`moduleDetection` controls whether TypeScript treats files as global scripts or isolated modules, independent of runtime output.**