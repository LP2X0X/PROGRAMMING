---
tags: 
 - typescript
 - compiler
---

## 1. What is `tsc`

- `tsc` is the command-line compiler for TypeScript.
    
- Running `tsc` without arguments looks for a `tsconfig.json` in the current directory (or parent directories) and compiles based on it. ([Tslang](https://tslang.com.cn/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript 中文文档：Documentation - tsc CLI Options"))
    
- You can also compile specific files directly:
    
    ```bash
    tsc index.ts
    tsc src/*.ts
    ```
    
    In that case the `tsconfig.json` is ignored. ([Tslang](https://tslang.com.cn/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript 中文文档：Documentation - tsc CLI Options"))
    
- It supports a **“build mode”** (with project references) as well. ([Tslang](https://tslang.com.cn/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript 中文文档：Documentation - tsc CLI Options"))
    

---

## 2. How configuration works

You configure compiler behavior either via flags on the command line (e.g., `tsc --target ES6 --module commonjs`) or via a `tsconfig.json` which has a `"compilerOptions"` section. ([TypeScript V2](https://typescript-v2-527-ortam.vercel.app/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Handbook - Compiler Options"))

In a React context, you’ll mostly put a `tsconfig.json` in your project root and set options relevant for JSX, React, module resolution, etc.

---

## 3. Key categories of compiler options (with relevant examples for React)

The options are many; I’ll group them and highlight the ones you’ll likely care about.

### 3.1 Basic emit / language settings

- `--target` : sets the JavaScript language version for output (e.g., `es5`, `es2015`, `esnext`). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--module` : sets module system for output (e.g., `commonjs`, `esnext`, `amd`, `umd`). Important for bundlers & React. ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--outDir` : where emitted JS files go. ([Runebook](https://runebook.dev/en/articles/typescript/compiler-options/compiler-options?utm_source=chatgpt.com "TypeScript - Unleashing the Power of tsc: Essential Compiler Options Explained"))
    
- `--rootDir` : the root folder of TS source files; ensures output paths mirror input. ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--jsx` : for React/JSX projects, tells TS how to handle `.tsx` files and JSX syntax (e.g., `react`, `react-jsx`, `preserve`). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    

### 3.2 Module resolution & path mapping

- `--moduleResolution` : how TS resolves module imports (e.g., `node`, `nodenext`, `classic`). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--baseUrl` & `--paths` : set base directory and alias mapping for modules (useful in React when you want `@components` etc). ([TypeScript V2](https://typescript-v2-527-ortam.vercel.app/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Handbook - Compiler Options"))
    

### 3.3 Strictness / type-checking controls

- `--strict` : enables a bundle of strict options (strict null checks, etc) → recommended in React for safety. ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--noImplicitAny`, `--noImplicitThis`, `--strictNullChecks`, `--strictFunctionTypes`, etc. ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--noUnusedLocals`, `--noUnusedParameters` : helps clean up unused code. ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    

### 3.4 Emit / declaration file options

- `--declaration` : generate `.d.ts` files (useful for library authors). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--emitDeclarationOnly` : emit only `.d.ts` and no JS. ([typescript.treinaweb.com.br](https://typescript.treinaweb.com.br/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--sourceMap`, `--inlineSourceMap`, `--inlineSources` : helpful for debugging in React (so you can map back to TSX). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    

### 3.5 Compile / watch / build options

- `--watch` : watch files and recompile on changes (handy for development). ([typescript.treinaweb.com.br](https://typescript.treinaweb.com.br/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--incremental` : enable incremental builds (faster subsequent builds). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--build` / `-b` : project references and multi-project builds. Probably less common in single-React-app. ([Tslang](https://tslang.com.cn/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript 中文文档：Documentation - tsc CLI Options"))
    

### 3.6 Misc / advanced flags

- `--esModuleInterop`, `--allowSyntheticDefaultImports` : for interoperability between CommonJS and ES modules (often needed if you import older packages). ([Runebook](https://runebook.dev/en/articles/typescript/compiler-options/compiler-options?utm_source=chatgpt.com "TypeScript - Unleashing the Power of tsc: Essential Compiler Options Explained"))
    
- `--forceConsistentCasingInFileNames` : ensures imports respect file name casing (useful on case-sensitive file systems). ([Runebook](https://runebook.dev/en/articles/typescript/compiler-options/compiler-options?utm_source=chatgpt.com "TypeScript - Unleashing the Power of tsc: Essential Compiler Options Explained"))
    
- `--skipLibCheck`, `--skipDefaultLibCheck` : skip type-checking of `.d.ts` libraries (can speed things up). ([TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html?utm_source=chatgpt.com "TypeScript: Documentation - tsc CLI Options"))
    
- `--noEmitOnError` : don’t produce output files if there are type errors. Good for CI. (you asked about this earlier) ([TypeScript Docs](https://typescriptdocs.com/project-config/compiler-options.html?utm_source=chatgpt.com "tsc CLI Options | Typescript Docs"))
    

---

## 4. Recommended `tsconfig.json` snippet for a React-centric project

Since you’re working with React and focusing on React only, here’s a sample `tsconfig.json` with important options:

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "esnext",
    "jsx": "react-jsx",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "noEmitOnError": true,
    "moduleResolution": "node",
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["components/*"]
    },
    "sourceMap": true
  },
  "include": ["src/**/*.ts", "src/**/*.tsx"],
  "exclude": ["node_modules", "dist"]
}
```

You can tailor many of these flags further, but this gives you a **good baseline** for React apps.

---

## 5. How to use `tsc`

- Run `tsc` → will pick up `tsconfig.json` and compile.
    
- For watch mode:
    
    ```bash
    tsc --watch
    ```
    
- For single file:
    
    ```bash
    tsc src/App.tsx
    ```
    
    (Note: this bypasses `tsconfig.json` defaults).
    
- Use `--project` (or `-p`) to specify a path to a `tsconfig.json`:
    
    ```bash
    tsc --project tsconfig.production.json
    ```
    

---

## 6. Things to be careful about with React

- Make sure `"jsx": "react-jsx"` (for React 17+ new JSX transform) or `"react"` (for older).
    
- If using React with `import React from 'react'`, check `"esModuleInterop": true` and `"allowSyntheticDefaultImports": true`.
    
- `moduleResolution` should match your bundler (e.g., webpack, Vite).
    
- Using path aliases (`baseUrl`, `paths`) requires your bundler to mirror them (if bundler doesn’t pick them up you’ll get runtime errors).
    
- Beware of `noEmitOnError`: in development you might want it off so you still get output even if there are some type errors—depends on your workflow.
    
- If you’re using `.css`/`.svg`/image imports, you’ll need module-declarations (`declare module "*.css";`) or enable `resolveJsonModule`/`allowJs` etc depending on setup.
    