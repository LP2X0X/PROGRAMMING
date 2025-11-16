---
tags: 
 - react
 - redux
 - method
---

- `prepare` is an **optional function** you can attach to a reducer in the `reducers` object of `createSlice`.
    
- It lets you **pre-process the action payload** (and optionally add a `meta` field) **before the reducer function runs**.
    
- This is useful if you want to **encapsulate logic for generating payloads**.
    
- It will return a new object which will then *contains* the payload object for the reducer.
    

---

# ⭐ Structure

```js
const slice = createSlice({
  name: 'example',
  initialState: { items: [] },
  reducers: {
    addItem: {
      reducer(state, action) {
        state.items.push(action.payload);
      },
      // The parameters here will corresponding to the one we will pass to the action creator "addItem"
      // If we don't use this function then the action creator will only accept one parameter which is the payload object
      prepare(name, quantity) {
        return {
          payload: {
            id: Date.now(),   // auto-generate ID
            name,
            quantity,
          },
          meta: {
            createdAt: new Date().toISOString()
          }
        };
      }
    }
  }
});
```

---

### How it works

1. When you dispatch:
    

```js
dispatch(slice.actions.addItem('Apple', 3));
```

2. RTK calls the **`prepare` function first**:
    

```js
prepare('Apple', 3)
```

- Returns:
    

```js
{
  payload: { id: 123456, name: 'Apple', quantity: 3 },
  meta: { createdAt: '2025-11-15T08:00:00Z' }
}
```

3. Then RTK calls the **reducer** with the action:
    

```js
reducer(state, { type: 'example/addItem', payload: {...}, meta: {...} })
```

- So the reducer sees a **ready-to-use payload**.
    

---

# ⭐ Key Points

- `prepare` is **only called when dispatching the action**.
    
- It allows you to:
    
    - Compute default values
        
    - Add metadata
        
    - Format or validate the payload
        
- The **reducer itself stays simple**, only consuming `action.payload`.
    

```ad-important
You can add [[Web Dev/5 - React/5.4 - How React Works/5.4.1 - Terms/Side Effect|side effect]] to the prepare function.
```

---

# ⭐ Example Usage

```js
const todoSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: {
      reducer(state, action) {
        state.push(action.payload);
      },
      prepare(title) {
        return {
          payload: {
            id: Date.now(),
            title,
            completed: false
          }
        };
      }
    }
  }
});

dispatch(todoSlice.actions.addTodo('Learn Redux Toolkit'));

// Action received by reducer:
{
  type: 'todos/addTodo',
  payload: { id: 123456, title: 'Learn Redux Toolkit', completed: false }
}
```

---

# ✅ Summary

|Feature|Purpose|
|---|---|
|`prepare`|Prepares the action payload (and optionally meta) before it reaches the reducer|
|`reducer`|Handles the prepared payload to update state|
|Use case|Auto-generate IDs, timestamps, or format payloads consistently|