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

# ☝️ The rules

![[Pasted image 20251111191133.png|center|700]]

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
    

![[Pasted image 20251111192624.png|center|700]]

```ad-note
The reason for the third case is that when the dependency state changes, the component re-renders once. After that, the `useEffect` runs, triggering another change to the dependent state — which causes the component to re-render a second time.
```

````ad-note
```js
  useEffect(
    function () {
      function playSound() {
        if (!allowSound) return;
        const sound = new Audio(clickSound);
        sound.play();
      }

      playSound();
    },
    [duration, allowSound]
  );
```
The dependency array in `useEffect` can include values that aren’t directly used inside the effect’s code. 
This is sometimes necessary for **business logic consistency** — for example, when you want the effect to re-run in response to certain state changes (like `duration` or `allowSound`), even if the effect’s function doesn’t explicitly use them. It ensures the effect stays in sync with the intended behavior of the app.
````

---

# Removing unnecessary dependencies

![[Pasted image 20251111192201.png|center|700]]

---


⚡ In short:  
The dependency array is **how you tell React when an effect or memoized value should update**. Think of it as a "watch list" — React keeps track of the variables, and re-runs the effect if they change.