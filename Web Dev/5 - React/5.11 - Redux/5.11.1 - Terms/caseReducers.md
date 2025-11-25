---
tags: 
 - react
 - redux
 - term
---

In Redux Toolkit, every slice created by `createSlice()` automatically generates an object called **`caseReducers`**.

It is simply an **internal map** that stores all the reducer functions you define inside the `reducers:` field.

Example slice:

```js
const counterSlice = createSlice({
  name: "counter",
  initialState: 0,
  reducers: {
    increment(state) {
      return state + 1;
    },
    add(state, action) {
      return state + action.payload;
    }
  }
});
```

RTK internally builds:

```js
caseReducers = {
  increment: (state, action) => { ... },
  add: (state, action) => { ... }
}
```

You normally **don’t touch `caseReducers` directly**, but RTK uses them under the hood when creating:

- the reducer function
    
- action creators
    
- action types
    

---

# 🧠 **Why `caseReducers` exist**

Because `createSlice()` needs:

1. The reducer functions (your logic)
    
2. The action names
    
3. A way to generate action types (`sliceName/actionName`)
    
4. A single final reducer that switches between actions
    

`caseReducers` stores all your tiny reducer functions in one place so RTK can:

- auto-generate actions
    
- build the final reducer
    
- provide metadata for devtools
    

---

# 📌 **When you might see `caseReducers`**

Usually when debugging or inspecting a slice object:

```js
console.log(counterSlice);
```

You may see:

```
{
  name: "counter",
  reducer: ƒ,
  actions: { increment, add },
  caseReducers: { increment: ƒ, add: ƒ }
}
```

This is just RTK exposing the raw reducer map.

---

# 📌 **Tip and Trick**

You can use caseReducer to **call another reducer’s logic** from inside a different reducer.

### ✔ Why this works

- All your reducers live inside `caseReducers`
    
- They all share the same `(state, action)` signature
    
- RTK doesn’t freeze or hide them — you can access them safely
    

```js
const todosSlice = createSlice({
  name: "todos",
  initialState: [],
  reducers: {
    addTodo(state, action) {
      state.push({ id: Date.now(), text: action.payload });
    },

    addTodoWithTrim(state, action) {
      // 1. Clean data
      const trimmed = action.payload.trim();

      // 2. Reuse the existing reducer!
      todosSlice.caseReducers.addTodo(state, { payload: trimmed });
    }
  }
});
```

- You **cannot** call actions — you call the _reducer functions_ directly.
    
- This works only **inside reducers** (not components).
    
- Pass a fake action object with the shape, or the action itself:
    
    ```js
    { payload: value }
    ```

---

# ⭐ **Summary**

**`caseReducers` = the internal object that holds all reducer functions you define inside a slice.**  
RTK uses it internally to generate actions and build the final reducer.  
You normally never call or modify it yourself.

---

If you want, I can also explain:

- how caseReducers are used to build the slice reducer
    
- caseReducers vs extraReducers
    
- what a "case reducer" means in Redux terminology