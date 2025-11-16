---
tags: 
 - react
 - redux
 - thunk
 - how
---

# ✅ **1. What Redux Thunk Does**

Middleware that lets you:

- Dispatch **functions** instead of plain objects
    
- Perform **async logic** (fetch, timers, side effects)
    
- Dispatch other **actions before/after** the async work
    

```js
dispatch(async (dispatch, getState) => { ... });
```

---

# ✅ **2. Using Redux Thunk with Classic Redux**

### **Step 1: Install**

```bash
npm install redux-thunk
```

### **Step 2: Add as middleware**

```js
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import rootReducer from "./rootReducer";

const store = createStore(rootReducer, applyMiddleware(thunk));
```

---

### **Step 3: Create a thunk**

```js
export function fetchUsers() {
  return async function (dispatch, getState) {
    dispatch({ type: "users/loading" });

    const res = await fetch("/api/users");
    const data = await res.json();

    dispatch({ type: "users/loaded", payload: data });
  };
}
```

### **Step 4: Use**

```js
dispatch(fetchUsers());
```

---

# 🚀 **3. Using Thunk with Redux Toolkit (RTK)**

### 🔥 RTK already includes Redux Thunk by default.

You **do not** need to install or configure anything.

---

## **(A) Manual Thunk in RTK**

```js
export const fetchUsers = () => async (dispatch, getState) => {
  dispatch(usersLoading());

  const res = await fetch("/api/users");
  const data = await res.json();

  dispatch(usersLoaded(data));
};
```

Usage:

```js
dispatch(fetchUsers());
```

---

## **(B) Using `createAsyncThunk` (recommended)**

### Step 1: Create thunk

```js
import { createAsyncThunk } from "@reduxjs/toolkit";

export const fetchUsers = createAsyncThunk(
  "users/fetchUsers",
  async () => {
    const res = await fetch("/api/users");
    return await res.json(); // returned value becomes `action.payload`
  }
);
```

---

### Step 2: Handle states in `extraReducers`

```js
const usersSlice = createSlice({
  name: "users",
  initialState: {
    list: [],
    status: "idle", // idle | loading | success | error
  },
  reducers: {},

  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = "loading";
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = "success";
        state.list = action.payload;
      })
      .addCase(fetchUsers.rejected, (state) => {
        state.status = "error";
      });
  },
});
```

---

### Step 3: Dispatch

```js
dispatch(fetchUsers());
```

---

# 🧠 **4. When Should You Use Thunks?**

Use thunks for:

- Fetching API data
    
- Reading from state before deciding what to do
    
- Writing async business logic outside components
    
- Avoiding data-fetching spread inside UI
    

---

# ⭐ **5. Core Patterns to Remember**

### **A thunk returns a function**

```js
const myThunk = () => (dispatch, getState) => { ... };
```

### **Dispatching inside a thunk**

```js
dispatch(actionCreator());
dispatch({ type: "something" });
```

### **Use getState**

```js
const token = getState().auth.token;
```

### **Errors**

Use `rejectWithValue` with `createAsyncThunk` for custom errors.

---

# 🔥 **6. Quick Comparison: Classic vs RTK**

|Feature|Classic Redux|Redux Toolkit|
|---|---|---|
|Setup|Must manually add middleware|Thunk included by default|
|Async actions|Write thunks manually|`createAsyncThunk()`|
|Reducers|Manual switch statement|`extraReducers` builder pattern|
|Boilerplate|High|Very low|