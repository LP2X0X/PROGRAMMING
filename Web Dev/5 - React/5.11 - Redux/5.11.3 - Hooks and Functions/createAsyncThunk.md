---
tags: 
 - react
 - redux
 - toolkit
 - function
---

- `createAsyncThunk` is a **utility from Redux Toolkit** for handling **async actions** (e.g., API calls).
    
- It **automatically generates action types** for the **three async states**:
    
    1. `pending` – the async process has started
        
    2. `fulfilled` – the async process succeeded
        
    3. `rejected` – the async process failed
        
- It **returns a thunk action creator** that you can dispatch like a normal action.
    

---

# ⭐ Syntax

```js
import { createAsyncThunk } from '@reduxjs/toolkit';

const fetchUsers = createAsyncThunk(
  'users/fetchUsers',   // action type prefix
  async (arg, thunkAPI) => {
    // async logic
    const response = await fetch(`/api/users`);
    return response.json(); // becomes action.payload in fulfilled case
  }
);
```

---

### Parameters:

1. **typePrefix** (`string`)
    
    - Used to generate action types automatically:
        
        - `'users/fetchUsers/pending'`
            
        - `'users/fetchUsers/fulfilled'`
            
        - `'users/fetchUsers/rejected'`
            
2. **payloadCreator** (`async function`)
    
    - Contains your async code
        
    - Arguments:
        
        - `arg` → the parameter you pass when dispatching
            
        - `thunkAPI` → an object with:
            
            - `dispatch` → can dispatch additional actions
                
            - `getState` → access current store state
                
            - `rejectWithValue` → customize rejected payload
                
            - `extra` → extra argument if configured
                

---

# ⭐ Example Usage

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, thunkAPI) => {
    try {
      const res = await fetch('/api/users');
      const data = await res.json();
      return data; // goes to fulfilled action.payload
    } catch (err) {
      return thunkAPI.rejectWithValue(err.message); // goes to rejected
    }
  }
);

// Slice
const usersSlice = createSlice({
  name: 'users',
  initialState: { list: [], loading: false, error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => { state.loading = true; state.error = null; })
      .addCase(fetchUsers.fulfilled, (state, action) => { state.loading = false; state.list = action.payload; })
      .addCase(fetchUsers.rejected, (state, action) => { state.loading = false; state.error = action.payload; });
  }
});

export default usersSlice.reducer;
```

---

# ⭐ How it works behind the scenes

1. You dispatch the thunk:
    

```js
dispatch(fetchUsers());
```

2. RTK automatically dispatches:
    

- `{ type: 'users/fetchUsers/pending' }` immediately
    
- Waits for the async function to finish:
    
    - Success → dispatch `{ type: 'users/fetchUsers/fulfilled', payload: result }`
        
    - Failure → dispatch `{ type: 'users/fetchUsers/rejected', payload: error }`
        

3. You handle these **three states in `extraReducers`**.
    

---

# ⭐ Key Benefits

- No need to manually write **pending / fulfilled / rejected actions**.
    
- Works with **`dispatch` and `getState`** inside the async function.
    
- Handles **errors** cleanly with `rejectWithValue`.
    
- Keeps **reducers simple**; all async logic is in the thunk.
    

---

# ⭐ Example with Arguments

```js
export const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId, thunkAPI) => {
    const res = await fetch(`/api/users/${userId}`);
    return res.json();
  }
);

// Dispatch:
dispatch(fetchUserById(123));
```

- `userId` becomes the `arg` in your async function.
    

---

✅ **Summary**

- `createAsyncThunk` = utility for async Redux logic
    
- Automatically generates **pending / fulfilled / rejected actions**
    
- Returns a thunk you can dispatch
    
- Use `extraReducers` to update state based on async action lifecycle
    
- Supports `thunkAPI` for `dispatch`, `getState`, and `rejectWithValue`
    