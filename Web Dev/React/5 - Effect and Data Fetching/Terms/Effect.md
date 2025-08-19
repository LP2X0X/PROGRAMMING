---
tags:
  - react
  - term
  - fundamental
---

The **`useEffect` hook** in React lets you run **side effects** in function components. Side effects are things React components can’t do during rendering, such as:

- Fetching data from an API
- Directly manipulating the DOM
- Setting up subscriptions or event listeners
- Starting/stopping timers

---

### Basic Syntax

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

---

### Key Concepts

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
    

---

### Common Use Cases

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
    

---

👉 Think of `useEffect` as:

- **Mount effect** → run once
- **Update effect** → run on dependency change
- **Cleanup effect** → run before unmount or re-run

---

![[Pasted image 20250818194802.png|center]]