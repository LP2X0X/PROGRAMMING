---
tags: 
 - react
 - note
---

### Why side effects must go in `useEffect`

React has two phases when rendering a component:

1. **Render (pure calculation phase)**
    
    - React calls your component function.
        
    - It expects the function to be _pure_: given the same props and state, it should return the same JSX.
        
    - This phase can run **many times** (due to StrictMode, concurrent rendering, or re-renders).
        
    - No changes to the outside world should happen here.
        
2. **Commit (side-effect phase)**
    
    - Once React has decided what to put in the DOM, it applies the changes.
        
    - After this point, React calls `useEffect` callbacks.
        
    - This is the _safe place_ to run code that interacts with the outside world.
        

---

### What counts as a side effect?

- Modifying the DOM directly
    
- Fetching data (network requests)
    
- Subscribing to events
    
- Setting timers
    
- Logging, analytics, etc.
    

Anything that **reaches outside of React’s render → JSX process** is considered a side effect.

---

### Why it matters

If you did side effects directly in the component body:

```jsx
function App() {
  fetch("/api/data"); // ❌ bad, runs every render
  return <div>Hello</div>;
}
```

- It would run on **every render**, even unnecessary ones.
    
- It could cause infinite loops (e.g., state updates trigger re-render → code runs again → re-render …).
    
- React might call your component function multiple times without committing the result (like in Concurrent Mode), meaning your effect could fire even if nothing actually changed in the UI.
    

---

### `useEffect` solves this

By using `useEffect`, you’re telling React:

> “Hey, once you’re done with the render and commit, _then_ run this code — and only when these dependencies change.”

That makes side effects:

- **Predictable** → tied to lifecycle (mount, update, unmount).
    
- **Safe** → won’t run during React’s render calculations.
    
- **Efficient** → runs only when needed.
    

---

👉 In short:  
Side effects go in `useEffect` because **the render phase must stay pure and idempotent**, and React needs to control _when_ effects happen so they don’t mess up rendering.