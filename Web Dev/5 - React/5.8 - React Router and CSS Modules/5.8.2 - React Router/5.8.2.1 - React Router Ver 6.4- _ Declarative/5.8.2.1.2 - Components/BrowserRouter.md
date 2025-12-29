---
tags: 
 - react
 - router
 - browser
---

### 🧭 **`<BrowserRouter>` Overview**

**Purpose:**  
Wraps your entire React app to enable **client-side navigation** using the browser’s **History API** — so routes change **without full page reloads**.

---

### 🧱 **Basic Syntax**

```jsx
import { BrowserRouter } from "react-router-dom";
import App from "./App";

<BrowserRouter>
  <App />
</BrowserRouter>
```

This provides routing context to everything inside (like `<Routes>` and `<Route>`).

---

### ⚙️ **What It Does**

- Listens to URL changes (`window.history.pushState`)
    
- Keeps UI in sync with the address bar
    
- Enables back/forward navigation
    
- Works seamlessly with `<Link>` and `<Route>`
    

---

### 🧩 **Typical Structure**

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./Home";
import About from "./About";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

### ⚠️ **Notes**

- Use **`BrowserRouter`** for modern web apps (pretty URLs like `/about`).
    
- Use **`HashRouter`** if you need support for older servers or static hosting (URLs like `/#/about`).
    
- Only **one BrowserRouter** should wrap your app.
    

---

✅ **In short:**  
`<BrowserRouter>` is the top-level component that **enables React Router** — it connects your routes to the browser’s URL and navigation system.