---
tags: 
  - react
  - term
  - effect 
  - fundamental
---

### ✅ Definition

The **dependency array** is the second argument you pass to React hooks like `useEffect`, `useMemo`, and `useCallback`. It tells React _when_ to re-run the effect or recompute the value, based on changes in its listed dependencies.

```jsx
useEffect(() => {
  // side effect code
}, [dep1, dep2]);
```

---

# 🛠 How it works

- React compares the values in the array **between renders**.
    
- If **any dependency has changed**, the effect runs again.
    
- If **no dependency has changed**, the effect is skipped (performance optimization).
    

---

# 📦 Common Cases

### 1. **No dependency array**

```jsx
useEffect(() => {
  console.log("Runs after every render");
});
```

➡️ Runs **after every render** (initial + updates).

---

### 2. **Empty dependency array `[]`**

```jsx
useEffect(() => {
  console.log("Runs once (after mount)");
}, []);
```

➡️ Runs **only once** when the component mounts (like `componentDidMount`).

---

### 3. **With dependencies**

```jsx
useEffect(() => {
  console.log("Runs when count changes");
}, [count]);
```

➡️ Runs **only when `count` changes** and **on mount**.

---

# 🔄 Cleanup with dependency array

If your effect returns a cleanup function, React will call it **before re-running** the effect (and on unmount):

```jsx
useEffect(() => {
  const id = setInterval(() => console.log("tick"), 1000);
  return () => clearInterval(id); // cleanup
}, [count]);
```

- Cleanup runs **before next effect run** when `count` changes.
    
- Also runs on component unmount.
    

---

# ⚠️ Gotchas

1. **Must include all external variables**
    
    - If you use variables from the component scope inside `useEffect`, they should be listed in the array.
        
    - Otherwise, you may run into **stale closures**.
        
    
    ```jsx
    useEffect(() => {
      console.log(count); // count must be in deps
    }, [count]);
    ```
    
2. **Stable references**  
    Functions/objects/arrays are new references on each render, so they can trigger re-runs unnecessarily. To avoid this, use `useCallback` or `useMemo`.
    
3. **Exhaustive-deps ESLint rule**  
    React recommends enabling this ESLint rule to catch missing dependencies.
    

---

# ✅ Best Practices

- Always include all variables you use inside the effect in the dependency array.
    
- Use `useCallback` / `useMemo` to stabilize functions/objects if needed.
    
- Use `[]` only when you are **sure** the effect must run only once (e.g., fetch on mount).
    

---

⚡ In short:  
The dependency array is **how you tell React when an effect or memoized value should update**. Think of it as a "watch list" — React keeps track of the variables, and re-runs the effect if they change.