---
tags: 
 - react
 - redux
 - option
 - object
---

# ⭐ `createSlice` Option Object

`createSlice` takes **one object argument** with specific properties. The basic syntax:

```js
import { createSlice } from '@reduxjs/toolkit';

const slice = createSlice({
  name: 'sliceName',
  initialState: {},          // required
  reducers: {},              // optional if using extraReducers
  extraReducers: {},         // optional
  // optional extra options (like 'devTools' config in some cases)
});
```

---

## 1️⃣ `name` (string) — required

- Unique name for the slice.
    
- Used to **prefix action types** generated from `reducers`.
    
- Example:
    

```js
name: 'counter'  // action type: 'counter/increment'
```

---

## 2️⃣ `initialState` — required

- Defines the **initial state** for this slice.
    
- Can be **any type**: object, array, primitive, or even a function that returns the initial state.
    

```js
initialState: { value: 0 }
```

---

## 3️⃣ `reducers` — optional but commonly used

- Object where each **key** is a **“slice reducer” function**.
    
- RTK automatically generates:
    
    - **Action types**: `<slice name>/<key>`
        
    - **Action creators**: `slice.actions.<key>()`
        
- Reducer function signature:
    

```ts
(state, action) => newState
```

- You can **mutate state directly** because RTK uses **Immer** internally.
    

**Example:**

```js
reducers: {
  increment(state) { state.value++; },
  addBy(state, action) { state.value += action.payload; }
}
```

- Auto-generated actions:
    

```js
slice.actions.increment() // { type: 'counter/increment' }
slice.actions.addBy(5)    // { type: 'counter/addBy', payload: 5 }
```

```ad-note
You can also use the [[prepare]] function for the reducer here.
```

---

## 4️⃣ `extraReducers` — optional

- Use when you want this slice to **handle actions defined elsewhere**, e.g., **actions from other slices** or **`createAsyncThunk` actions**.
    

**Two forms:**

### Object notation

```js
extraReducers: {
  [someAction.type]: (state, action) => { ... },
}
```

### Builder callback notation (recommended for TS)

```js
extraReducers: (builder) => {
  builder
    .addCase(someAction, (state, action) => { ... })
    .addCase(someAsyncThunk.pending, (state) => { ... })
    .addCase(someAsyncThunk.fulfilled, (state, action) => { ... });
}
```

- **Builder notation** is safer because it avoids typos in string action types.
    
- Perfect for **async actions** with `createAsyncThunk`.
    

---

## 5️⃣ Optional extra options

- `initialState` can be a function for dynamic initialization.
    
- Other options are rarely needed, but you can extend slices for **middleware or dev tools**.
    

---

### ✅ Full Example

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetch',
  async () => {
    const res = await fetch('/api/users');
    return res.json();
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    list: [],
    loading: false,
    error: null,
  },
  reducers: {
    clearUsers(state) {
      state.list = [];
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.list = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export const { clearUsers } = usersSlice.actions;
export default usersSlice.reducer;
```

---

### ✅ Summary of `createSlice` Option Object

|Property|Required|Purpose|
|---|---|---|
|`name`|Yes|Slice name, used as action type prefix|
|`initialState`|Yes|Initial state of the slice|
|`reducers`|No|Local reducer functions, generate actions automatically|
|`extraReducers`|No|Handle external actions (other slices, async thunks)|
|others|No|Rare, advanced use cases|