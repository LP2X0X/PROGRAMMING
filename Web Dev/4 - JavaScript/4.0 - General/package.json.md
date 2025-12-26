---
tags: 
 - js
 - term
 - general
---

It’s the **manifest file** for a Node.js / JavaScript project. It tells Node (and npm/yarn/pnpm) things like:

- What your project is called
    
- Which version it is
    
- Which dependencies it needs
    
- What scripts you can run
    
- Metadata for publishing to npm
    

Basically, it’s the "project contract" for your app or library.

---

## 🔑 Common Sections in `package.json`

### 1. **Basic Info**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A simple demo app",
  "author": "Your Name",
  "license": "MIT"
}
```

- **name**: project/package name (must be lowercase, no spaces if published).
    
- **version**: semantic version (`major.minor.patch`).
    
- **description**: short summary.
    
- **author**: your name/email.
    
- **license**: license type (MIT, ISC, etc.).
    

---

### 2. **Scripts**

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "server": "json-server --watch data/cities.json --port 9000 --delay 500"
}
```

- Defines custom commands run with `npm run <script>` (or `yarn <script>`).
    
- Example: `npm run dev` runs `vite`.
    

So `"server": "json-server ..."` means you can spin up your fake API with `npm run server`.

---

### 3. **Dependencies**

```json
"dependencies": {
  "react": "^18.3.0",
  "react-dom": "^18.3.0"
}
```

- Packages your project needs in **production**.
    
- Installed when someone runs `npm install`.
    

---

### 4. **Dev Dependencies**

```json
"devDependencies": {
  "vite": "^5.1.0",
  "eslint": "^8.57.0"
}
```

- Only needed in **development** (e.g., build tools, linters, test libs).
    
- Won’t be included in production builds.
    

---

### 5. **Other Common Fields**

- **main**: entry point file for libraries (`index.js` usually).
    
- **module**: ES module entry point (for bundlers like webpack, rollup).
    
- **type**: `"module"` (for ES modules) or `"commonjs"`.
    
- **engines**: Node/npm versions required.
    
- **browserslist**: target browsers for frontend builds.
    

---

### 6. **Example Full File**

Here’s a realistic `package.json` for a React + Vite project with json-server:

```json
{
  "name": "cities-dashboard",
  "version": "1.0.0",
  "description": "A demo app to explore React Router and json-server",
  "author": "Your Name",
  "license": "MIT",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "server": "json-server --watch data/cities.json --port 9000 --delay 500"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "react-router-dom": "^6.22.0"
  },
  "devDependencies": {
    "vite": "^5.1.0",
    "eslint": "^8.57.0",
    "json-server": "^0.17.4"
  }
}
```

---

✅ **In short:**

- **Metadata**: name, version, description, license, author.
    
- **Scripts**: shortcuts for dev/build/test.
    
- **Dependencies**: packages for runtime.
    
- **DevDependencies**: packages for development only.
    
- **Extra fields**: entry points, config, engines, etc.
    