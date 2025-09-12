---
tags: 
 - react
 - term
 - fundamental
---

## 🔹 What is a Custom Hook?

A **custom hook** is just a **JavaScript function** that:

- Starts with the word `use` (so React can enforce [[Hooks#**☝️ Rules of hooks**|rules of hooks]]).
    
- Calls other React hooks inside (like `useState`, `useEffect`, `useRef`, etc.).
    
- Encapsulates reusable logic so you don’t repeat it in multiple components.
    

It’s not a new feature — just a convention + function composition.


![[Pasted image 20250911194456.png|center|500]]

---

## 🔹 Why use them?

- **Reusability** → Move stateful logic into a function you can reuse across components.
    
- **Cleaner components** → Extract side-effect logic so UI stays focused on rendering.
    
- **Abstraction** → Hide complex implementation details from components.
    

---

## 🔹 Example 1: useWindowSize

```jsx
import { useState, useEffect } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size; // 👈 expose state
}
```

Usage:

```jsx
function MyComponent() {
  const { width, height } = useWindowSize();
  return <p>Window size: {width} x {height}</p>;
}
```

---

## 🔹 Example 2: useLocalStorage

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

Usage:

```jsx
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage("theme", "light");

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      Switch to {theme === "light" ? "dark" : "light"}
    </button>
  );
}
```

---

## 🔹 Rules to Remember

1. **Name must start with `use`** → `useSomething()`.
    
2. **Only call hooks inside React functions** (component or custom hook).
    
3. **Don’t call hooks conditionally** → order must be the same every render.
    

---

✨ So, a custom hook is basically:

> “A function that packages up hook logic so you can reuse it like a building block.”