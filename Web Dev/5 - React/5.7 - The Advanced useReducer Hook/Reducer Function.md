---
tags: 
 - react
 - function
 - fundamental
---

A **reducer** is simply a **pure function** that:

```
(state, action) → newState
```

It receives:

- the **current state**
    
- an **action** (an object describing _what happened_)
    

and returns:

- a **new state**
    

It never mutates the original state.

---

# 🌿 Reducer in **React** (useReducer)

In React’s `useReducer` hook, the reducer function manages **local component state**.

### Example reducer:

```js
function counterReducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      throw new Error("Unknown action");
  }
}
```

Then you use it like:

```js
const [state, dispatch] = useReducer(counterReducer, { count: 0 });
```

### Key ideas (React):

- Reducer is just a function.
    
- React calls it when you call `dispatch()`.
    
- You return a _new_ state (no mutation).
    
- Perfect for **complex local state** or **state with many transitions**.
    

---

# 🏢 Reducer in **React-Redux**

In Redux, reducers define **global app state updates**.

They follow the **same signature** as React reducers:

```
(state, action) → newState
```

But Redux reducers:

- usually manage _slices_ of state
    
- are combined with other reducers (`combineReducers`)
    
- must always return _some_ state (even initial state)
    

### Example Redux reducer:

```js
function todosReducer(state = [], action) {
  switch (action.type) {
    case "todo/add":
      return [...state, action.payload];
    case "todo/remove":
      return state.filter(t => t.id !== action.payload);
    default:
      return state; // never throw
  }
}
```

### Key ideas (Redux):

- Reducer must be **pure**.
    
- It cannot:
    
    - mutate state
        
    - do async work
        
    - read/write external variables
        
- It **always** returns a value.
    

---

# 🔍 Differences Between React Reducer vs Redux Reducer

|Feature|`useReducer` (React)|Redux Reducer|
|---|---|---|
|Scope|Local component state|Global app state|
|Initial State|passed when calling useReducer|defined inside reducer|
|Must return initial state|No|Yes|
|Can throw error|Yes (usually if unknown action)|No (unknown actions must return existing state)|
|Used with middleware|No|Yes (thunk, saga…)|
|Combined into root reducer|No|Yes|

---

# 💡 Why Is It Called a “Reducer”?

Because it follows the same pattern as `Array.reduce()`:

```js
array.reduce((prevState, action) => newState, initialState)
```

Redux internally does something similar — it reduces over dispatched actions to compute the next state.

---

# 🔑 The Essence

A **reducer function** is:

> A pure function that calculates the **next state** of your app/component based on a **current state** and an **action**.

It provides:

- predictability
    
- testability
    
- centralized logic for all state changes
    
