---
tags: 
 - react
 - redux
 - hook
---

# ⭐ What is `useDispatch`

- `useDispatch` is a **hook** provided by **React-Redux**.
    
- It gives you access to the **Redux store’s `dispatch` function** inside a React component.
    
- `dispatch` is used to **send actions** to your Redux store, which then **update the state via reducers**.
    

---

# ⭐ Syntax

```js
import { useDispatch } from 'react-redux';

const dispatch = useDispatch();
```

- `dispatch` is a function: `dispatch(action)`
    
- You can call it with:
    
    - **Plain action objects** (classic Redux)
        
    - **Thunk functions** (async logic)
        
    - **RTK-generated actions** (`createSlice` or `createAsyncThunk`)
        

---

# ⭐ How to Use

### 1. Dispatch a plain action

```js
dispatch({ type: 'counter/increment' });
```

### 2. Dispatch an action creator

```js
import { increment, addBy } from './counterSlice';

dispatch(increment());    // RTK action creator
dispatch(addBy(5));
```

### 3. Dispatch a thunk (async function)

```js
import { fetchUsers } from './userActions';

dispatch(fetchUsers());   // Thunk handles async logic
```

---

# ⭐ Why it works

- In React-Redux, `Provider` puts the Redux store in **React context**.
    
- `useDispatch()` reads the store from the context and returns the `dispatch` function.
    
- This allows **any component** to send actions **without passing dispatch down as a prop**.
    

---

# ⭐ Best Practices

1. **Keep components “dumb”**: dispatch actions, don’t modify state directly.
    
2. **Use action creators** instead of hardcoding action objects — less error-prone.
    
3. **Thunks or async actions**: always dispatch from `useDispatch` if side effects are needed.
    
4. **Memoize callbacks** if you pass them down to child components (optional):
    

```js
const handleClick = useCallback(() => dispatch(increment()), [dispatch]);
```

5. **Don’t call dispatch in render** — only in event handlers, effects, or thunks.
    

---

# ⭐ Example in a Component

```jsx
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, addBy } from './counterSlice';

function Counter() {
  const value = useSelector(state => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Count: {value}</p>
      <button onClick={() => dispatch(increment())}>+1</button>
      <button onClick={() => dispatch(addBy(5))}>+5</button>
    </div>
  );
}

export default Counter;
```

---

✅ **Summary**

- `useDispatch` gives you the **store’s dispatch function** inside React.
    
- You call `dispatch(action)` to update the Redux state.
    
- Works with **plain actions**, **action creators**, and **thunks**.
    
- Allows components to trigger state changes **without directly accessing reducers**.
    