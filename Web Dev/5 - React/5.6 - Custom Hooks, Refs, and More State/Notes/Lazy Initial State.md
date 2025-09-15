---
tags: 
 - react
 - term
---

In React, **lazy initial state** means you can initialize state with a function so that the function only runs once — on the initial render — instead of on every render.

This is useful when the initial state is expensive to compute (e.g., parsing data, reading from localStorage).

---

### Example without lazy initialization

```jsx
import { useState } from "react";

function App() {
  const [count, setCount] = useState(expensiveComputation());

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}

function expensiveComputation() {
  console.log("Expensive function ran");
  return 10;
}
```

Here, `expensiveComputation()` runs **every render**, not just the first one.

---

### Example with lazy initialization

```jsx
import { useState } from "react";

function App() {
  const [count, setCount] = useState(() => expensiveComputation());

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}

function expensiveComputation() {
  console.log("Expensive function ran");
  return 10;
}
```

Here, React will **call the function only once**, on the initial mount, and reuse that value for the state. On re-renders, it won’t re-run `expensiveComputation()`.

---

✅ **Rule of thumb**:

- Use `useState(initialValue)` when `initialValue` is cheap.
    
- Use `useState(() => initialValue)` when `initialValue` is expensive to compute.
    