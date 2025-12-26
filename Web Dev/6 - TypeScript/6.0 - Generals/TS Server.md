---
tags: 
 - typescript
 - general
---

**`tsserver`** is the **TypeScript Language Service** that runs in the background to provide editor and IDE features for TypeScript _and_ JavaScript projects.

It is **not a compiler** and **not a runtime**. It is a long-running Node.js process used by editors.

---

## What `tsserver` does

`tsserver` powers features such as:

- IntelliSense / auto-completion
    
- Type checking and error diagnostics
    
- Go to definition / find references
    
- Hover type information
    
- Refactoring (rename symbol, extract, etc.)
    
- Quick fixes and code actions
    

This applies to:

- `.ts`, `.tsx`
    
- `.js`, `.jsx` (with type inference / JSDoc)
    
- `.d.ts`
    

---

## How it fits in the TypeScript toolchain

| Tool           | Purpose                          |
| -------------- | -------------------------------- |
| `tsc`          | Compile TypeScript → JavaScript  |
| `tsserver`     | Provide editor language features |
| Node / Browser | Execute JavaScript               |

Editors **do not** call `tsc` continuously. They talk to `tsserver`.

---

## How editors use `tsserver`

1. Your editor starts `tsserver` (Node.js process)
    
2. The editor sends JSON-based requests over stdio:
    
    - “What is the type of this symbol?”
        
    - “Give me completions here”
        
3. `tsserver` analyzes:
    
    - `tsconfig.json`
        
    - Project files
        
    - Type definitions
        
4. It returns structured responses the editor renders as UI
    

Examples of editors using `tsserver`:

- VS Code (built-in)
    
- Vim / Neovim
    
- Emacs
    
- Sublime Text
    
- WebStorm (partially; JetBrains has its own engine but integrates TS)
    

---

## Why `tsserver` exists (design rationale)

Running `tsc` repeatedly would be:

- Too slow
    
- Stateless
    
- Not incremental
    

`tsserver`:

- Keeps an in-memory model of your project
    
- Reuses type information
    
- Updates incrementally as you type
    

That is why IntelliSense feels “live”.

---

## Relation to `tsconfig.json`

`tsserver` uses `tsconfig.json` to determine:

- Module resolution
    
- Target / lib
    
- Strictness flags
    
- Project boundaries
    

Misconfigured `tsconfig` → bad editor diagnostics  
Correct `tsconfig` → accurate suggestions

---

## JavaScript support (important)

Even in **plain JS projects**, `tsserver` is often active:

```js
// @ts-check
function add(a, b) {
  return a + b;
}
```

It infers types and reports errors without compilation.

---

## Common misconceptions

❌ “tsserver compiles my code”  
→ No. `tsc` compiles.

❌ “tsserver runs in the browser”  
→ No. It is a Node.js process.

❌ “TypeScript only works for `.ts` files”  
→ `tsserver` also understands `.js`.

---

## When you would care about `tsserver`

You typically interact with it **indirectly**, but you care when:

- IntelliSense is slow or wrong
    
- Errors differ between editor and `tsc`
    
- Monorepos or project references behave oddly
    
- You configure `typescript.tsserver.*` settings in VS Code
    

---

### One-sentence summary

> **`tsserver` is the persistent TypeScript language engine that editors use to understand your code and provide smart tooling—separate from compilation and execution.**
