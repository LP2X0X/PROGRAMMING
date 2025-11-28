---
tags: 
 - react
 - redux
 - thunk
---

Under the hood, `createAsyncThunk` _does_ use the Promise `.then()` / `.catch()` pattern to automatically dispatch the corresponding actions for each lifecycle state.

It **does not rely on `try/catch` at the outer level** in the generated wrapper — it wraps your async function in a promise chain so it can dispatch:

- `pending`
    
- `fulfilled`
    
- `rejected`
    

at the correct times.

---

# ✅ How `createAsyncThunk` actually works internally

Your async payload creator:

```js
export const fetchUser = createAsyncThunk(
  "users/fetchUser",
  async (id) => {
    const res = await fetch(`/users/${id}`);
    return res.json();
  }
);
```

Redux Toolkit generates a **thunk function** that runs something like this internally:

```js
function generatedThunk(arg) {
  return function thunk(dispatch, getState) {

    // 1. Dispatch pending
    dispatch({ type: "users/fetchUser/pending", meta: { arg } });

    // 2. Call your async function
    const promise = payloadCreator(arg, { dispatch, getState });

    // 3. Use .then() / .catch() to handle the result
    return promise
      .then((result) => {
        dispatch({
          type: "users/fetchUser/fulfilled",
          payload: result,
          meta: { arg }
        });
        return result;
      })
      .catch((error) => {
        dispatch({
          type: "users/fetchUser/rejected",
          error: error instanceof Error ? error : String(error),
          meta: { arg }
        });
        throw error;
      });
  };
}
```

✔ Yes: this uses `.then()` and `.catch()`  
✔ Yes: this is how RTK knows when your Promise resolves or rejects  
✔ Yes: this is how the three auto-generated actions are dispatched

---

# 🧠 Why `.then()` and `.catch()`?

Two reasons:

### **1. A thunk function cannot use `await` at the top level**

The outer thunk is _not_ async — it's synchronous and must return a function.

So `.then()` / `.catch()` is the cleanest way to attach lifecycle actions.

### **2. It supports any return value**

Even if your payload creator doesn't use `async`, but returns:

- a Promise
    
- a thunk
    
- a synchronous value
    

The Promise chain still works.

---

# ⭐ Summary (exact and correct)

> **createAsyncThunk wraps your async payload creator in a promise chain (`then` / `catch`) so it can automatically dispatch pending, fulfilled, and rejected actions based on the promise outcome.**