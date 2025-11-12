---
tags: 
 - react
 - effect
 - note
 - closure
---

### Prerequisite

[[Closure]]
[[Lexical Environment]]

---

## 🧠 Closures and `useEffect`

### 1. **How closure affects `useEffect`**

- When this function was first created, it _closed over_ the state and props that existed during the initial render — the same moment the function itself was defined — and never re-created in subsequent renders. In other words, it captured a snapshot of the state and props at their initial values.
- But, any function that was created at the initial render and then not recreated still has access to that initial snapshot of states and props.

> Therefore, it will never have access to the latest state or props — it will always reference the initial snapshot (*stale closure*) captured when the function was first created.

```jsx
useEffect(function () {
  document.title = `Your ${number}-exercise workout`;
}, []);
```


---

### 2. **How to fix stale closure**

To make sure your effect always uses the latest value, you must **include dependencies**:

```jsx
useEffect(function () {
  document.title = `Your ${number}-exercise workout`;
}, [number]);
```

```ad-note
Specifying the dependency array is like telling React, “I know you can’t automatically see the latest values during each render, but you only need to re-run this effect when these specific values change.”
```

---

### 3. **Key takeaway**

- Each render = **new closures** and **new effect callback**.
    
- If your effect depends on a value, **it must be listed in the dependency array**.
    
- Otherwise, the effect may use **stale data** (the closure trap).
    
- Use the **functional state update form** (`setState(prev => ...)`) to safely reference latest state inside async logic.
    

---

### 4. **Summary Table**

|Concept|Description|Fix|
|---|---|---|
|Closure|Function keeps variables from its defining render|Include dependencies or use functional update|
|Stale data|Effect reads old values|Add missing deps in array|
|Functional update|Safely access latest state inside closure|`setState(prev => prev + 1)`|
|Dependency array|Tells React which values the effect depends on|Keep it accurate and complete|