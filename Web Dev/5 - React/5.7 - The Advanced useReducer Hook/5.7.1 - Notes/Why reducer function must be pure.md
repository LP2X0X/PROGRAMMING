---
tags: 
 - react
 - useReducer
 - reducer
 - function
---

### 🧠 What does “pure function” mean?

A **pure function** is one that:

1. **Always returns the same output** given the same input.
    
2. **Has no side effects** — it doesn’t modify things outside of itself (like the DOM, network requests, or global variables).
    

Example of a **pure** function:

```js
function add(a, b) {
  return a + b;
}
```

Example of an **impure** function:

```js
function addToCart(item) {
  cart.push(item); // ❌ modifies external variable
}
```

---

### ⚙️ Why reducer functions _must_ be pure

Reducers are the **core of predictable state management** — both in React’s `useReducer` and Redux.  
Here’s why purity is essential:

1. **🧩 Predictability**
    
    - If a reducer is pure, you can always predict the next state:
        
        ```js
        nextState = reducer(currentState, action);
        ```
        
        This always gives the same result, making your app’s state logic **reliable and testable**.
        
2. **🧪 Easier debugging and testing**
    
    - Pure reducers can be tested in isolation, since they don’t depend on the outside world.
        
3. **🔁 Time travel & undo/redo features**
    
    - Libraries like Redux DevTools record actions and replay them — this only works if reducers behave the same every time (pure).
        
4. **⚡ Performance optimizations**
    
    - React can skip unnecessary re-renders if it knows the reducer doesn’t have hidden side effects.
        

---

### 🚫 What counts as a “side effect”

Inside a reducer, you should **never**:

- Mutate state directly:
    
    ```js
    state.count++ // ❌
    ```
    
- Call asynchronous code:
    
    ```js
    fetch('/api') // ❌
    ```
    
- Modify external variables or browser storage (`localStorage`, `sessionStorage`, etc.)
    
- Trigger UI updates (like `alert()` or `document.querySelector()`)
    

---

### ✅ Reducer Example (Pure)

```js
function counterReducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      return state;
  }
}
```

### ❌ Reducer Example (Impure)

```js
function counterReducer(state, action) {
  if (action.type === "increment") {
    localStorage.setItem("count", state.count + 1); // ❌ side effect
    return { count: state.count + 1 };
  }
}
```

---

### 🧭 TL;DR

Reducers must be **pure** because React (and Redux) depend on them being:

- **Predictable**
    
- **Reproducible**
    
- **Free of side effects**
    

That’s what allows React to manage state efficiently and consistently.