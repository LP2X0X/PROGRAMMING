---
tags: 
 - react
 - redux
 - toolkit
---

# ⭐ What Redux Toolkit Is

**Redux Toolkit (RTK)** is the **official, recommended** way to write Redux code today.  
It replaces old boilerplate (`createStore`, `switch-case` reducers, manual actions, immutable updates) with **simple APIs**, best practices **built-in**, and strong TypeScript support.

Think of RTK as the **“batteries-included Redux”**.

![[Pasted image 20251115152848.png|center|700]]

---

# ⭐ Installation & Setup (Redux Toolkit + React-Redux)

Redux Toolkit works **together** with React-Redux. Therefore, you must install **both**:

Run this in your React project:
```bash
npm install @reduxjs/toolkit react-redux
```

That’s all — no extra packages like redux-thunk (already included) and no need for redux-devtools-extension (RTK configures DevTools for you).

---

# ⭐ Why RTK Exists (the problems it solves)

Traditional Redux had problems:

|Old Redux Problem|RTK Fix|
|---|---|
|Too much boilerplate (action types, creators, switch-case)|`createSlice` auto-generates actions + reducer|
|Hard immutable updates (deep spread copies)|**Immer** included → write “mutating” code safely|
|Hard to set up store (middleware, DevTools)|`configureStore` sets all defaults correctly|
|Async logic complicated|`createAsyncThunk` simplifies async flows|
|Common mistakes (mutating state, forgetting DevTools)|RTK defaults prevent mistakes|

RTK’s job = **make Redux simpler, safer, and shorter**.

---

# ⭐ Core APIs You Must Know

Below are the _four_ APIs that cover 95% of what you will ever do.

---

## 1. `configureStore()`

Creates the store with the best default setup:

- Automatically combine reducers
    
- Redux DevTools enabled
    
- Thunk middleware included
    
- Serializability and immutability checks
    
- No need for `createStore`, `applyMiddleware`, `compose`
    

✔ **Example**

```js
const store = configureStore({
// This used to be the "rootReducer" in old Redux
  reducer: {
    user: userReducer,
    todos: todosReducer,
  },
});
```

---

## 2. `createSlice()`

The heart of RTK — defines:

- Initial state
    
- Reducers
    
- Automatically generated action creators
    
- Automatically generated action types
    

It uses Immer internally → allows **mutating syntax**.

✔ **Example**

```js
const counterSlice = createSlice({
  name: "counter",
  initialState: { value: 0 },
  reducers: {
    increment(state) {
      state.value++;       // looks like mutation but safe
    },
    addBy(state, action) {
      state.value += action.payload;
    }
  }
});

export const { increment, addBy } = counterSlice.actions;
export default counterSlice.reducer;
```

```ad-note
- Each key in the `reducers` object **corresponds to a reducer function**, not directly to an action type.
    
- **RTK automatically creates the action types** using `<slice name>/<key>` for you.
    
- It also **creates action creators** with the same names (`increment()`, `addBy(payload)`).
```

```ad-tip
We can get the action creators from the slice object's actions property to debug if we want.
```

```ad-note
If we do nothing to the state, we can return nothing (in case of other reducer, we must return the original state at least).
```

---

## 3. `createAsyncThunk()`

The modern way to handle async work (API calls).

- Handles `pending` / `fulfilled` / `rejected`
    
- Generates action types automatically
    
- Works perfectly with `extraReducers`
    

✔ **Example**

```js
export const fetchUser = createAsyncThunk(
  "user/fetchUser",
  async (id) => {
    const res = await fetch(`/api/users/${id}`);
    return res.json();
  }
);
```

In the slice:

```js
extraReducers: (builder) => {
  builder
    .addCase(fetchUser.pending, (state) => {
      state.loading = true;
    })
    .addCase(fetchUser.fulfilled, (state, action) => {
      state.loading = false;
      state.data = action.payload;
    });
}
```

---

## 4. `createSelector()` (from Reselect, included)

Efficient derived data + prevents re-renders.

✔ Example:

```js
export const selectCompleted = createSelector(
  (state) => state.todos,
  (todos) => todos.filter(t => t.done)
);
```

---

# ⭐ How Redux Toolkit Fits into React

You still use:

```jsx
import { Provider } from "react-redux";
import { useSelector, useDispatch } from "react-redux";
```

✔ React usage:

```jsx
function Counter() {
  const value = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch(increment())}>
      {value}
    </button>
  );
}
```

RTK does not replace React-Redux — it just provides a better Redux setup.

---

# ⭐ Folder Structure (Recommended)

```
src/
  app/
    store.js
  features/
    counter/
      counterSlice.js
    todos/
      todosSlice.js
```

---

# ⭐ What RTK Automates for You

RTK automatically handles:

### 1. **Immutable updates** via Immer

→ write `state.value++` and RTK converts it to safe immutable code.

### 2. **Action types**

→ auto-generated: `"counter/increment"`, `"counter/addBy"`

### 3. **Action creators**

→ automatically created from reducer names.

### 4. **Thunk setup**

→ no need to import `redux-thunk`.

### 5. **DevTools integration**

→ enabled by default.

### 6. **Middleware configuration**

→ includes safety checks (help catch bugs).

---

# ⭐ When To Use RTK vs Context / useReducer

|When Redux Toolkit is good|When Context/useReducer is enough|
|---|---|
|Large app / many components|Small app|
|Shared global state|Few props|
|Complex updates|Simple state|
|Async logic|No async|
|Need DevTools, time-travel|Not needed|

If your data grows or becomes async → RTK is far better.

---

# ⭐ Common RTK Patterns

### Pattern 1: "Slice per feature"

One slice for each domain: `authSlice`, `cartSlice`, `todoSlice`.

### Pattern 2: Reuse thunks like service functions

All async logic in thunks.

### Pattern 3: Normalized state

RTK encourages storing data like a DB:

```js
{
  entities: { 1: {...} },
  ids: [1]
}
```

RTK even has `createEntityAdapter` for this.

---

# ⭐ Cheat Sheet Summary

- Use **`configureStore`** → store setup
    
- Use **`createSlice`** → reducers + actions
    
- Use **`createAsyncThunk`** → async logic
    
- Use **`useSelector` + `useDispatch`** in React
    
- RTK gives you **cleaner code, less boilerplate, and immutable-by-default state** with mutation-like syntax
    