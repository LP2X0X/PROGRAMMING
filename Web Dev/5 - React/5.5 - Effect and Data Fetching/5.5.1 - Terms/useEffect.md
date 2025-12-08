---
tags:
  - react
  - term
  - hook
  - effect
  - fundamental
---

The **`useEffect` hook** in React lets you run **side effects** in function components. Side effects are things React components can’t do during rendering, such as:

- Fetching data from an API
- Directly manipulating the DOM
- Setting up subscriptions or event listeners
- Starting/stopping timers

```ad-note
Effect are used to keep a component synchronised with some external system.
```

![[Pasted image 20250827174352.png|center|600]]

```ad-tip
It can also helps synchronize states.
```

---

## Basic Syntax

```jsx
import { useEffect, useState } from "react";

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Component rendered or count changed");

    return () => {
      console.log("Cleanup before effect runs again or before unmount");
    };
  }, [count]); // dependency array
}
```

```ad-warning
You must return a callback function, not call it.
```

---

## Key Concepts

[[Dependency Array]]

1. **No dependency array (`[]` missing)**
    
    ```jsx
    useEffect(() => {
      console.log("Runs after every render");
    });
    ```
    
2. **Empty dependency array (`[]`)**
    
    ```jsx
    useEffect(() => {
      console.log("Runs only once after initial render (mount)");
    }, []);
    ```
    
3. **With dependencies (`[value]`)**
    
    ```jsx
    useEffect(() => {
      console.log("Runs when `value` changes");
    }, [value]);
    ```
    
4. **Cleanup function** (for unmounting or before re-running effect)
    
    ```jsx
    useEffect(() => {
      const id = setInterval(() => {
        console.log("Interval running...");
      }, 1000);
    
      return () => clearInterval(id); // cleanup
    }, []);
    ```

![[Pasted image 20250827174253.png|center|600]]

---

## Common Use Cases

- **Fetching Data**
    
    ```jsx
    useEffect(() => {
      fetch("/api/data")
        .then(res => res.json())
        .then(data => console.log(data));
    }, []);
    ```
    
- **Listening to Window Resize**
    
    ```jsx
    useEffect(() => {
      const handleResize = () => console.log(window.innerWidth);
      window.addEventListener("resize", handleResize);
    
      return () => window.removeEventListener("resize", handleResize);
    }, []);
    ```
    
- **Updating Document Title**
    
    ```jsx
    useEffect(() => {
      document.title = `Count: ${count}`;
    }, [count]);
    ```
    

```ad-important
Each effect should do **only one thing**! Use **one useEffect hook for each side effect**. This makes effects easier to [[Cleanup Function|clean up]].
```

---

👉 Think of `useEffect` as:

- **Mount effect** → run once
- **Update effect** → run on dependency change
- **Cleanup effect** → run before unmount or re-run

---

## useEffect run asynchronously **after component render**

### 🔄 Render + Commit Phases in React

1. **Render phase**
    
    - React calls your component function(s).
        
    - It calculates what the DOM _should_ look like.
        
    - No DOM updates happen yet.
        
    - This is **pure and synchronous**.
        
2. **Commit phase**
    
    - React updates the actual DOM.
        
    - Browser paints the UI on the screen.
        
    - User sees the update.
        
3. **Effects phase** (`useEffect`)
    
    - After the browser has painted, React runs your side effects asynchronously (queued in the task/microtask queue).
        
    - That way, your UI is visible immediately, _before_ slow or blocking side effects run.
        

### ⚡ Key Point:

- `useEffect` always runs **after paint** (asynchronously, in the background).
    
- `useLayoutEffect`, in contrast, runs **synchronously after DOM mutations but before paint**.
    
    - That’s why `useLayoutEffect` can block the paint if it’s heavy.
        

### Example:

```jsx
useEffect(() => {
  console.log("✅ Runs AFTER paint");
});

useLayoutEffect(() => {
  console.log("⏱ Runs BEFORE paint (blocks UI)");
});
```

Order if you update state:

```
Render → Commit DOM → Browser paints → useEffect
                          ↑
                        useLayoutEffect runs here
```
