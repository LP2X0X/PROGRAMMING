---
tags: 
 - typescript
 - vite
 - setup
---

### Setup Notes: Run Vite Dev Server + TypeScript Checking in Parallel (`run-p dev:*`)

Goal: **Vite serves/builds fast**, while **TypeScript checks types in the background**, so the dev server never waits for `tsc`.

---

## 1) Install the parallel runner

Use `npm-run-all` (provides `run-p`):

```bash
npm i -D npm-run-all
```

---

## 2) Ensure TypeScript is installed (for type-checking)

```bash
npm i -D typescript
```

---

## 3) Add scripts to `package.json`

```json
{
  "scripts": {
    "dev": "run-p dev:*",
    "dev:vite": "vite",
    "dev:tsc": "tsc --watch",
    "build": "tsc --noEmit && vite build"
  }
}
```

What each does:

- **`dev`**: runs all scripts matching `dev:*` in parallel
    
- **`dev:vite`**: starts the Vite dev server (transpiles TS via esbuild)
    
- **`dev:tsc`**: runs TypeScript type-checking continuously
    
- **`build`**: blocks production build if type errors exist
    

---

## 4) Confirm `tsconfig.json` exists

Create one if you don’t have it:

```bash
npx tsc --init
```

Common minimum expectation for Vite + React TS projects:

- `jsx` set appropriately (often `react-jsx`)
    
- `noEmit` can be true or false; for dev we are not using TS emit anyway
    

---

## 5) Run development

```bash
npm run dev
```

Expected behavior:

- Terminal shows **two processes**
    
    - Vite starts immediately (fast)
        
    - `tsc --watch` prints type errors as you code
        
- Type errors **do not stop** the dev server or HMR
    

---

## 6) How to read what you see in the terminal

- Vite logs: server URL, HMR updates, build warnings
    
- `tsc --watch` logs: “Found X errors” / “Watching for file changes”
    

If you see TS errors:

- Your app may still run (because transpilation continues)
    
- But you should fix them before `build` / merging code
    

---

## 7) Recommended optional add-ons

### A) Only type-check (no serve)

```bash
npm run dev:tsc
```

### B) Only run the dev server

```bash
npm run dev:vite
```

---

## Summary mental model

- **Vite** = “rebuild/serve quickly (no type-check)”
    
- **tsc --watch** = “type-check continuously (no rebuild output)”
    
- *_run-p dev:_ ** = “run both at the same time”
    

</br>
</br>

----

</br>
</br>

Vite does **not** use `tsc` to transform TypeScript to JavaScript because `tsc` is **too slow** and does **type-checking**, which Vite deliberately separates from transpilation.

Here is **exactly how Vite does it**.

---

## Short answer

> **Vite uses ESBuild to transpile TypeScript → JavaScript, without type-checking.**

---

## The build pipeline (step by step)

### 1️⃣ Dev server

- Vite starts a dev server
    
- When the browser requests a `.ts` / `.tsx` file:
    
    - Vite passes it to **esbuild**
        
    - esbuild **strips types** and outputs JS
        
    - The JS is served instantly
        

⚡ This is why Vite is fast.

---

### 2️⃣ Production build

- Vite uses:
    
    - **esbuild** → TS → JS
        
    - **Rollup** → bundling, tree-shaking, code splitting
        

So:

```text
TypeScript ──esbuild──▶ JavaScript ──Rollup──▶ Bundle
```

---

## What ESBuild does (and does NOT do)

### ✅ What it does

- Removes TypeScript syntax
    
- Transforms TS/TSX to valid JS
    
- Respects `tsconfig.json` for:
    
    - `target`
        
    - `jsx`
        
    - `jsxImportSource`
        
    - `paths` (partially)
        

### ❌ What it does NOT do

- ❌ Type checking
    
- ❌ Emit `.d.ts` files
    
- ❌ Advanced TS-only transforms
    

---

## Where type-checking happens (if at all)

Vite expects you to run **type-checking separately**.

Common options:

### Option 1: Run `tsc --noEmit`

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}
```

### Option 2: Use `vite-plugin-checker`

- Runs `tsc` in a worker
    
- Does not block dev server
    

---

## Why Vite made this choice

|Tool|Purpose|
|---|---|
|`tsc`|Type analysis + emit|
|`esbuild`|Fast syntax transform|

Vite’s philosophy:

> **Transpile fast, check types separately**

This avoids:

- Blocking HMR
    
- Slow cold starts
    
- Double work during dev
    

---

## Comparison table

|Tool|Transpiles TS|Type-checks|Speed|
|---|---|---|---|
|`tsc`|✅|✅|Slow|
|Babel|✅|❌|Medium|
|**esbuild**|✅|❌|**Very fast**|

---

## Important nuance

Even though Vite does not use `tsc`:

- Your editor (VS Code) still uses `tsserver`
    
- Errors still appear while typing
    
- Build speed stays fast
    

---

## Final takeaway

> **Vite uses esbuild for TypeScript transpilation and completely separates type-checking from transformation.**  
> This design is the key reason Vite’s dev experience is so fast.