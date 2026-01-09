---
tags: 
 - npm
 - command
---

The `npm install` command (commonly aliased as `npm i`) is the workhorse of the Node.js ecosystem. Its primary job is to fetch code from the npm registry and place it into your project's `node_modules` folder so you can use it.

Here is a detailed breakdown of its different modes, flags, and behaviors.

---

### 1. The Two Main Modes

#### A. Installing Existing Dependencies (No Arguments)

When you run the command by itself in a project folder:

```bash
npm install
```

- **What it does:** It looks for a `package.json` file in your directory.
    
- **Behavior:** It installs **all** dependencies listed in that file.
    
- **Lockfile Priority:** If a `package-lock.json` exists, `npm` tries to match the exact versions listed there. If the `package.json` and `package-lock.json` disagree, `npm install` generally updates the lockfile to match the `package.json`.
    

#### B. Installing New Packages (With Arguments)

When you provide a package name:

```bash
npm install lodash
```

- **What it does:** It downloads the latest version of `lodash` from the npm registry.
    
- **Updates Files:**
    
    1. Downloads the code to `node_modules/lodash`.
        
    2. Adds `"lodash"` to the `dependencies` list in `package.json`.
        
    3. Records the exact version tree in `package-lock.json`.
        

---

### 2. Dependency Types (Where does it go?)

You usually don't want all packages treated the same (e.g., you need `react` for your app to run, but you only need `typescript` to build it). You control this with flags:

| **Flag**      | **Short** | **Meaning**                                                                                           | **Where it goes in package.json** |
| ------------- | --------- | ----------------------------------------------------------------------------------------------------- | --------------------------------- |
| **(Default)** |           | **Production Dependency:** Required for the app to run.                                               | `"dependencies"`                  |
| `--save-dev`  | `-D`      | **Dev Dependency:** Only needed for development/testing (e.g., Jest, ESLint).                         | `"devDependencies"`               |
| `--global`    | `-g`      | **Global Install:** Installs to your system, not the project folder (e.g., CLI tools like `nodemon`). | _Nowhere in project_              |
| `--no-save`   |           | **Temporary:** Installs to `node_modules` but does **not** add it to `package.json`.                  | _Nowhere_                         |

---

### 3. Version Control

You can specify exactly which version you want using the `@` symbol:

- `npm install react@18.2.0` (Exact version)
    
- `npm install react@latest` (The absolute newest tag)
    
- `npm install react@next` (The upcoming beta/alpha version)
    

---

### 4. `npm install` vs. `npm ci`

This is a critical distinction for modern development, especially in CI/CD pipelines (like GitHub Actions or Jenkins).

| **Feature**  | **npm install**                          | **npm ci (Clean Install)**                                   |
| ------------ | ---------------------------------------- | ------------------------------------------------------------ |
| **Goal**     | For developers editing code.             | For automated build servers.                                 |
| **Lockfile** | Can **modify** `package-lock.json`.      | **Strictly follows** `package-lock.json`. Error if mismatch. |
| **Folder**   | Tries to update existing `node_modules`. | Deletes `node_modules` and starts fresh.                     |
| **Speed**    | Slower (calculates trees).               | Faster (installs exactly what is locked).                    |

> **Best Practice:** Use `npm install` on your laptop. Use `npm ci` on your build server.

---

### 5. Troubleshooting Flags

Sometimes installation fails due to conflicts. These flags help you bypass strict checks:

- **`--legacy-peer-deps`**: The most common "fix" for modern npm errors. It tells npm to ignore conflicts between "peer dependencies" (common when mixing old and new React libraries).
    
- **`--force`**: Forces npm to fetch remote resources even if a local copy exists on disk, or tries to overwrite conflicting binaries.
    
- **`--production`**: Installs _only_ regular `dependencies` and ignores `devDependencies`. (Automatic in some deployment environments like Heroku).
    

---

### 6. Behind the Scenes (How it works)

When you type `npm install`, the process follows a specific lifecycle:

1. **Resolution:** npm checks `package.json` and queries the registry to build a "dependency tree" (calculating which versions of sub-dependencies work together).
    
2. **Fetching:** It downloads the `.tar.gz` files for the packages. It often checks a local machine cache (`~/.npm`) first to speed this up.
    
3. **Extraction:** It unpacks the files into `node_modules`.
    
4. **Linking:** It creates "bin links" (executable shortcuts) inside `node_modules/.bin` for packages that have CLI commands (like `webpack` or `tsc`).
    
