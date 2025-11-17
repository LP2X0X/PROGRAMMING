---
tags: 
 - react
 - redux
 - hook
---

# ⭐ What is `useSelector`

- `useSelector` is a **React hook** provided by **React-Redux**.
    
- It lets your React component **read a specific piece of state** from the Redux store.
    
- Unlike `store.getState()`, it **subscribes** to the store and will **re-render the component when the selected state changes**.
    

---

# ⭐ Syntax

```js
import { useSelector } from 'react-redux';

const value = useSelector((state) => state.someSlice.someValue);
```

- The function you pass (`state => ...`) is called a **selector**.
    
- It receives the **entire Redux state** and should return the part your component needs.
    

---

# ⭐ Example

```jsx
import React from 'react';
import { useSelector } from 'react-redux';

function Counter() {
  const count = useSelector(state => state.counter.value);

  return <p>Count: {count}</p>;
}

export default Counter;
```

- If `state.counter.value` changes in the store, the `Counter` component **automatically re-renders**.
    

---

# ⭐ Why it works

- `useSelector` internally **subscribes** to the Redux store.
    
- It compares the **previous selected value** with the **new value** after every dispatch.
    
- If the value changed (using strict equality `===`), it triggers a **re-render**.
    
- If the value is the same, it **does not re-render**, optimizing performance.
    

---

# ⭐ Best Practices

1. **Select only what you need**
    
    ```js
    // Good: only count
    const count = useSelector(state => state.counter.value);
    
    // Avoid selecting the whole slice if you only need one value
    const counterSlice = useSelector(state => state.counter); // may trigger extra renders
    ```
    
2. **Use memoized selectors for derived data**
    
    - Use `reselect` or `createSelector` if you compute derived data from state.
        
3. **Don’t mutate the state in selectors** — always treat it as **read-only**.
    
4. **Stable function reference**
    
    - You can define the selector outside the component for performance, especially in large apps.
        

---

# ⭐ Difference from `store.getState()`

|`useSelector`|`store.getState()`|
|---|---|
|React hook|Store method|
|Subscribes and triggers re-render|Snapshot only, no re-render|
|Should be used inside components|Can be used anywhere you have store|
|Selector receives state|Returns full state|

---

# ⭐ Quick Example with Dispatch

```jsx
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment } from './counterSlice';

function Counter() {
  const count = useSelector(state => state.counter.value);
  const dispatch = useDispatch();

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>+1</button>
    </>
  );
}
```

- `useSelector` reads state
    
- `useDispatch` updates state
    
- React re-renders automatically when selected state changes
    

---

✅ **Summary**

- `useSelector` = **React hook to read Redux state**
    
- Auto re-renders component when selected state changes
    
- Works alongside `useDispatch`
    
- Always select the **minimal slice of state** your component needs
    