---
tags: 
 - typescript
 - general
 - config
---

### 🔹 What Is `tsconfig.json`?

`tsconfig.json` is a configuration file that tells the **TypeScript compiler (`tsc`)**:

- What files to compile
    
- How to compile them
    
- Where to put the compiled output
    

It defines **compiler options**, **include/exclude patterns**, and **project references**.

---

### 🧩 Basic Example

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

## ⚙️ Sections of `tsconfig.json`

### 1. **compilerOptions**

The most important section — defines how TypeScript compiles your code.

---

### 🧱 **Core Options**

|Option|Description|Example / Tip|
|---|---|---|
|`target`|ECMAScript version output|`"ES5"`, `"ES6"`, `"ESNext"`|
|`module`|Module system to use|`"commonjs"`, `"esnext"`, `"amd"`|
|`lib`|Built-in type definitions to include|`["ES2020", "DOM"]`|
|`allowJs`|Allow JS files to be compiled|`true` for migration projects|
|`checkJs`|Type-check JS files|Works only if `allowJs` is true|
|`jsx`|JSX support mode|`"react"`, `"react-jsx"`, `"preserve"`|
|`outDir`|Output directory for compiled JS|`"./dist"`|
|`rootDir`|Base directory for input files|`"./src"`|
|`removeComments`|Strip comments from output|`"true"`|
|`noEmit`|Compile type-check only (no output)|`"true"`|
|`noEmitOnError`|Prevent emitting if there are errors|`"true"`|
|`sourceMap`|Generate `.map` files for debugging|`"true"`|

---

### 🔒 **Strict Type Checking Options**

These improve type safety and catch bugs early.

|Option|Description|Tip|
|---|---|---|
|`strict`|Enables all strict mode checks|Best to keep `true`|
|`noImplicitAny`|Error when type inferred as `any`|Forces explicit typing|
|`strictNullChecks`|Disallows assigning `null`/`undefined`|Prevents “billion dollar mistake”|
|`strictFunctionTypes`|Ensures function parameter types are compatible|Useful for callbacks|
|`strictBindCallApply`|Checks arguments for `.bind`, `.call`, `.apply`|Helps avoid silent errors|
|`alwaysStrict`|Emit JS with `"use strict"`|Recommended|

---

### 🧩 **Module Resolution Options**

|Option|Description|Example|
|---|---|---|
|`moduleResolution`|How TS looks for modules|`"node"`, `"classic"`|
|`baseUrl`|Base folder for module imports|`"./src"`|
|`paths`|Map import paths to specific folders|`{ "@utils/*": ["utils/*"] }`|
|`rootDirs`|Virtual directories for resolution|`["src", "generated"]`|
|`typeRoots`|Directories to look for `@types`|`["./types", "./node_modules/@types"]`|
|`types`|Limit included type definitions|`["node", "jest"]`|
|`resolveJsonModule`|Allow importing `.json`|`"true"`|

💡 **Tip:**  
Use `baseUrl` + `paths` to clean up ugly relative imports like `../../../utils`.

---

### 📦 **Emit / Output Control**

|Option|Description|
|---|---|
|`declaration`|Generate `.d.ts` files for libs|
|`declarationMap`|Generate source maps for declarations|
|`emitDeclarationOnly`|Only emit `.d.ts` files|
|`downlevelIteration`|Support for `for...of` and spread in ES5|
|`importHelpers`|Use `tslib` for helpers (reduces size)|
|`isolatedModules`|Ensures each file can be compiled alone (for Babel, etc.)|
|`esModuleInterop`|Fixes `import` issues with CommonJS|
|`allowSyntheticDefaultImports`|Allows `import React from "react"` even without default export|

💡 **Tip:**  
If you use Babel or bundlers (like Vite, Webpack, Next.js), enable `isolatedModules` and `esModuleInterop`.

---

### 🧪 **Experimental Options**

|Option|Description|Example|
|---|---|---|
|`experimentalDecorators`|Enable decorators syntax|`"true"`|
|`emitDecoratorMetadata`|Emit metadata for decorators|`"true"`|
|`useDefineForClassFields`|Change how class fields are emitted|`"true"` (aligns with ES spec)|

---

### 🧹 **Code Quality / Error Options**

|Option|Description|
|---|---|
|`noUnusedLocals`|Error on unused local variables|
|`noUnusedParameters`|Error on unused parameters|
|`noImplicitReturns`|Error if not all code paths return|
|`noFallthroughCasesInSwitch`|Warn on missing `break` in switch|
|`forceConsistentCasingInFileNames`|Enforce consistent file name casing|

---

### 2. **include**

Defines which files or directories to **include**.

```json
"include": ["src/**/*"]
```

---

### 3. **exclude**

Defines which files to **ignore**.

```json
"exclude": ["node_modules", "dist"]
```

If omitted, defaults to:  
`["node_modules", "bower_components", "jspm_packages"]`.

---

### 4. **files**

Explicitly list which files to compile.

```json
"files": ["src/main.ts", "src/config.ts"]
```

⚠️ **Tip:** Rarely used; prefer `include` unless your project is tiny.

---

### 5. **extends**

Inherit configuration from another `tsconfig.json`.

```json
{
  "extends": "./base.json",
  "compilerOptions": {
    "outDir": "./build"
  }
}
```

Useful for **monorepos** or **multiple environments** (e.g., `tsconfig.base.json`, `tsconfig.test.json`).

---

### 6. **references**

Used in **Project References** for large or monorepo setups.

```json
{
  "references": [{ "path": "../shared" }]
}
```

This allows incremental compilation across multiple projects.

---

## 🧠 Common Config Patterns

### ✅ Node.js Project

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

---

### ✅ React Project (Vite / CRA)

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```

---

### ✅ Library / SDK Project

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "ESNext",
    "declaration": true,
    "emitDeclarationOnly": false,
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src"]
}
```

---

## 🧩 Bonus: How `tsc` Uses It

- Run `tsc --init` → creates default `tsconfig.json`
    
- Run `tsc` → compiles based on `tsconfig.json`
    
- Run `tsc -p path/to/config` → use a custom config file
    
- Run `tsc --showConfig` → see the **effective** config after inheritance
    

---

## 🪄 Quick Tips

|Situation|Recommended Option|
|---|---|
|You use React|`"jsx": "react-jsx"`|
|You use Node.js|`"module": "commonjs"`|
|You use modern bundlers (Vite, Webpack 5+)|`"module": "esnext"`|
|You import JSON|`"resolveJsonModule": true`|
|You have slow builds|`"skipLibCheck": true"`|
|You want only type-checking|`"noEmit": true"`|
|You need safety|`"strict": true"`|

---

## ✅ Summary

|Category|Key Options|
|---|---|
|Target & Output|`target`, `module`, `outDir`, `rootDir`|
|Type Checking|`strict`, `noImplicitAny`, `strictNullChecks`|
|Module Resolution|`baseUrl`, `paths`, `moduleResolution`|
|Code Quality|`noUnusedLocals`, `noImplicitReturns`|
|Project Setup|`include`, `exclude`, `extends`, `references`|