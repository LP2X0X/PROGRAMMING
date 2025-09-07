---
tags: 
 - react
 - effect
 - function
---

The **cleanup function** in a React `useEffect` is a function you return from inside your effect. It’s used to clean up side effects (like subscriptions, timers, or event listeners) before the component re-renders or unmounts.

- It runs on two different occasions:
	- Before the effect is **executed again**
	- After a component **has unmounted** (notice the "has" word)

```ad-note
By using [[Closure|closure]], the clean up function can remember the variable inside the useEffect body.
```
---

### 1. **Basic Example**

```jsx
useEffect(() => {
  console.log("Effect runs");

  return () => {
    console.log("Cleanup runs");
  };
}, []);
```

- The effect (`console.log("Effect runs")`) runs **once** after mount (because of `[]`).
    
- The cleanup (`console.log("Cleanup runs")`) runs **before unmount**.
    

---

### 2. **On Re-render (dependencies)**

```jsx
useEffect(() => {
  console.log("Effect runs with count");

  return () => {
    console.log("Cleanup before next effect");
  };
}, [count]);
```

- Runs effect **after every change in `count`**.
    
- Cleanup runs **before the effect runs again**, preventing duplicate subscriptions or leaks.
    

---

### 3. **Typical Use Cases**

- **Event listeners**
    
    ```jsx
    useEffect(() => {
      const handleResize = () => console.log("Resized!");
      window.addEventListener("resize", handleResize);
    
      return () => {
        window.removeEventListener("resize", handleResize);
      };
    }, []);
    ```
    
- **Timers**
    
    ```jsx
    useEffect(() => {
      const id = setInterval(() => console.log("Tick"), 1000);
    
      return () => clearInterval(id);
    }, []);
    ```
    
- **Subscriptions**
    
    ```jsx
    useEffect(() => {
      const sub = someAPI.subscribe(data => setData(data));
    
      return () => sub.unsubscribe();
    }, []);
    ```
    

---

✅ **Key idea:** Cleanup ensures you don’t leave behind “dangling” effects (memory leaks, multiple subscriptions, runaway timers).

---

### Summary
![[Pasted image 20250907115624.png|center|600]]