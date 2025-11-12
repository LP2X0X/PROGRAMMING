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

- The function below, when it was first created it close over the states and props at the time it was created. That was in the initial render  

```jsx
useEffect(function () {
  document.title = `Your ${number}-exercise workout`;
}, []);
```

- The closure inside `setInterval` captured `count = 0` from the first render.
    
- So every second, it keeps adding `1` to `0`, never seeing updated values.
    

---

### 2. **How to fix stale closure**

To make sure your effect always uses the latest value, you must **include dependencies**:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1); // ✅ use functional update form
  }, 1000);
  return () => clearInterval(id);
}, []); // ✅ safe, because functional update doesn’t depend on closure
```

or if you actually need the current `count` in the effect logic:

```jsx
useEffect(() => {
  console.log("count changed:", count);
}, [count]); // ✅ re-run effect whenever count changes
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