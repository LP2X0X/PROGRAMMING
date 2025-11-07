---
tags: 
 - typescript
 - installation
---

## 🧠 1. Install TypeScript

### ✅ Global install (for `tsc` command)

```bash
npm install -g typescript
```

Check version:

```bash
tsc -v
```

---

### ✅ Local install (recommended per project)

```bash
npm install --save-dev typescript
```

Then you can run `npx tsc` instead of global `tsc`.

---

## ⚙️ 2. Initialize TypeScript in your project

```bash
npx tsc --init
```

This creates a **`tsconfig.json`** file — the main configuration file for TypeScript.

---

## 📄 3. Basic Project Structure

```
my-project/
├── src/
│   └── index.ts
├── dist/
├── package.json
└── tsconfig.json
```

---

## 🧩 4. Example `tsconfig.json`

Here’s a good starting point:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "noEmitOnError": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

## 🏗️ 5. Compile TypeScript

### Compile once:

```bash
npx tsc
```

### Watch mode (auto recompile on save):

```bash
npx tsc --watch
```

---

## 🧱 6. Run TypeScript directly (no manual compile)

Install `ts-node`:

```bash
npm install --save-dev ts-node
```

Then run:

```bash
npx ts-node src/index.ts
```

Great for small scripts or testing.

---

## 🧩 7. Typical `scripts` in package.json

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts"
  }
}
```

---

## 🧠 8. Extra Tools

|Tool|Purpose|
|---|---|
|`ts-node`|Run `.ts` files directly|
|`@types/node`|Type definitions for Node.js|
|`eslint` + `@typescript-eslint`|Lint TypeScript code|
|`vite` / `webpack` / `esbuild`|Build and bundle TypeScript apps|

---

## 🚨 9. Common Flags

|Flag|Meaning|
|---|---|
|`--noEmitOnError`|Don’t emit `.js` files if there are type errors|
|`--watch`|Recompile on changes|
|`--project <path>`|Use a specific `tsconfig.json`|
|`--outDir`|Output directory for compiled JS|
|`--rootDir`|Where your `.ts` files are located|

---

## 🧭 10. Quick Reference Summary

```bash
# Create a project
mkdir my-ts-app && cd my-ts-app
npm init -y

# Install TypeScript
npm install --save-dev typescript ts-node @types/node

# Create tsconfig.json
npx tsc --init

# Build
npx tsc

# Run directly
npx ts-node src/index.ts
```