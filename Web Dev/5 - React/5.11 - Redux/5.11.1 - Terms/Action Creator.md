---
tags: 
 - react
 - redux
 - convention
 - term
---

### **What is an action creator?**

An **action creator** is simply a function that returns an action object.  
It helps avoid manually writing `{ type, payload }` everywhere and keeps your code consistent.

---

# ⭐ Why use action creators?

- Ensures consistent action shapes
    
- Avoids typos in action types
    
- Makes code easier to refactor
    
- Works perfectly with Redux Toolkit slices
    
- Improves readability in components
    

---

# 📌 Classic Redux Example

```js
function addTodo(text) {
  return {
    type: 'todos/add',
    payload: text
  };
}
```

Usage:

```js
dispatch(addTodo('Buy milk'));
```

---

# 📌 Modern Redux (Redux Toolkit) Example

Redux Toolkit **generates action creators for you** inside a slice.

```js
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    add(state, action) {
      state.push({ id: nanoid(), text: action.payload });
    },
    remove(state, action) {
      return state.filter(todo => todo.id !== action.payload);
    }
  }
});

export const { add, remove } = todosSlice.actions;
```

Usage:

```jsx
dispatch(add('Buy milk'));
dispatch(remove(3));
```

RTK automatically creates:

```js
{ type: 'todos/add', payload: 'Buy milk' }
{ type: 'todos/remove', payload: 3 }
```

---

# 🎯 Key Points to Remember

- Action creators return **plain objects** (actions).
    
- They **describe what happened**, not how state changes.
    
- Components should **dispatch action creators**, not raw objects.
    
- With Redux Toolkit, you rarely write action creators manually—they’re generated for you.
    