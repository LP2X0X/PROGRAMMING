---
tags: 
 - react
 - optimize
 - bundle
---

### Prerequisite
[[Bundle]]
[[Code Splitting]]
[[Dynamic Imports]]

---

Here’s a **detailed explanation of code splitting in React using `React.lazy()` and dynamic `import()` in Vite**.

---

## 🧩 1. What is Code Splitting?

By default, Vite (and most bundlers) combines your app into one big **bundle**.  
**Code splitting** means **loading parts of your app only when they’re needed** — reducing the initial load time.

In React, the easiest way to achieve this is with **`React.lazy()` + `Suspense`** and **dynamic `import()`**.

---

## ⚙️ 2. Why use it?

✅ Faster initial load  
✅ Users download code only for the page they visit  
✅ Great for large SPAs with many routes or components

---

## 🧠 3. How it works conceptually

- `React.lazy()` lets you load a component _lazily_ (only when rendered).
    
- `import()` dynamically loads the component file at runtime.
    
- Vite automatically creates **separate JS chunks** for each lazy-loaded component.
    

---

## 💻 4. Example: Basic setup in Vite + React

### 📂 Project structure

```
src/
 ├─ components/
 │   ├─ About.jsx
 │   ├─ Home.jsx
 ├─ App.jsx
 └─ main.jsx
```

---

### `App.jsx`

```jsx
import React, { Suspense, lazy } from "react";

// Lazy-load components
const Home = lazy(() => import("./components/Home"));
const About = lazy(() => import("./components/About"));

function App() {
  return (
    <div>
      <h1>React Code Splitting Example</h1>

      <Suspense fallback={<p>Loading...</p>}>
        {/* Lazy components will load only when rendered */}
        <Home />
        <About />
      </Suspense>
    </div>
  );
}

export default App;
```

---

### `Home.jsx`

```jsx
export default function Home() {
  return <h2>🏠 Home Page</h2>;
}
```

### `About.jsx`

```jsx
export default function About() {
  return <h2>ℹ️ About Page</h2>;
}
```

---

## 🚀 5. How it behaves in Vite

When you run:

```bash
npm run build
```

Vite will automatically split the code like this:

```
dist/assets/index.[hash].js      <-- main bundle
dist/assets/About.[hash].js      <-- loaded when About renders
dist/assets/Home.[hash].js       <-- loaded when Home renders
```

So each component becomes a **separate chunk**, loaded **on demand**.

---

## ⚡ 6. Real-world example: with React Router

You can also code-split at **route level**:

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { Suspense, lazy } from "react";

const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));

export default function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<p>Loading page...</p>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

➡️ Each route loads its JS chunk **only when visited**.

---

## 🧩 7. Summary

|Concept|Description|
|---|---|
|**Code splitting**|Load parts of the app only when needed|
|**`React.lazy()`**|Declares a component that’s loaded via dynamic import|
|**`Suspense`**|Displays a fallback (like a loader) while waiting|
|**Vite**|Automatically splits code into separate chunks|
|**Benefit**|Faster load, smaller bundle, better UX|