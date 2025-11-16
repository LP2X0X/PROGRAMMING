---
tags: 
 - react
 - redux
 - method
---

# ⭐ What is `store.getState()`

- `store.getState()` is a **method on the Redux store object**.
    
- It returns the **current state tree** of the Redux store at the exact moment it’s called.
    
- Unlike `useSelector` in React, it **does not subscribe** to changes. It’s just a snapshot of state.
    

---

# ⭐ Syntax

```js
const currentState = store.getState();
```

- `store` is the object you create with `createStore` or `configureStore`.
    
- Returns the **full state object**, exactly as in the store.
    

---

# ⭐ Example

```js
import { createStore } from 'redux';

const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    default:
      return state;
  }
}

const store = createStore(counterReducer);

console.log(store.getState()); // { count: 0 }

store.dispatch({ type: 'increment' });

console.log(store.getState()); // { count: 1 }
```

- You can see the state **changes after dispatch**, but `getState()` itself **does not trigger re-renders** in React.
    

---

# ⭐ When to use `store.getState()`

1. **Outside React components** (e.g., in scripts, thunks, middleware).
    
2. **For debugging or logging state**.
    
3. **To read current state in non-react code** (e.g., when manually dispatching actions).
    

---

# ⭐ Difference from `useSelector`

|`store.getState()`|`useSelector`|
|---|---|
|Direct store method|React hook|
|Returns current state **snapshot only**|Subscribes to store changes & triggers re-render when value changes|
|Can be used anywhere with access to `store`|Must be used inside React function component|
|No automatic re-render|React component updates automatically if selected state changes|

---

# ⭐ Best Practices

- Avoid calling `store.getState()` inside React components — use `useSelector` instead.
    
- Use `store.getState()` mainly for **middleware, thunks, or external scripts**.
    
- Do **not mutate** the object returned by `getState()`; always treat it as **read-only**.
    

---

✅ **Summary**

- `store.getState()` = snapshot of the current store state.
    
- Useful outside React for reading state.
    
- Does not subscribe or cause re-renders.
    