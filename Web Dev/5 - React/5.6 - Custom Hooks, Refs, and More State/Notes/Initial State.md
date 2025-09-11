---
tags: 
 - react
 - note
---

React only looks at the **initial state value** you pass into `useState` **on the very first render** of a component.

---

### How it works

```jsx
const [count, setCount] = useState(() => expensiveCalculation());
```

- **On the first render**:
    
    - React calls the initializer (`expensiveCalculation()` here, or just the value you pass).
        
    - That return value is stored in React’s internal state storage (in the fiber linked list of hooks).
        
    - React associates it with the hook’s position in the component.
        
- **On re-renders**:
    
    - React does **not** call the initializer again.
        
    - It simply retrieves the stored state value from its internal linked list.
        
    - The function argument you originally passed to `useState` is ignored after the first render.
        

---

### Example

```jsx
function Counter() {
  const [count, setCount] = useState(() => {
    console.log("Initializer runs");
    return 0;
  });

  return (
    <button onClick={() => setCount(c => c + 1)}>
      {count}
    </button>
  );
}
```

- When the component mounts → `"Initializer runs"` logs once.
    
- Every click → only `setCount` updates state; the initializer never runs again.
    

---

### Why this design?

1. **Performance** → Avoids recomputing expensive initialization on every render.
    
2. **Consistency** → Keeps state stable between renders; only updates triggered by `setState` matter.
    

---

✅ So yes: React will **only use the initial state value on the initial render**. On later renders, it always takes the stored state instead.