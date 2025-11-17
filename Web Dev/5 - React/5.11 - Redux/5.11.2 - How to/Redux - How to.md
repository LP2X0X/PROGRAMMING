---
tags: 
 - react
 - redux
 - how
---

# ✅ **How to Use Redux — Step-by-Step**

Even though classic Redux is rarely used today, you may still want to understand the fundamentals. Below is the complete flow using **actions → reducer → store → provider → useSelector/useDispatch**.

---

## **0️⃣ Installation**

Run this in your React project for Redux:

```bash
npm install redux
```

And this for React-Redux:

```bash
npm install react-redux
```

---

## **1️⃣ Define Action Types**

Action types are just strings that describe **what happened** in the app.

```ad-note
Below are the old conventions, modern Redux user do not use this anymore. We just use the string in lower case. I keep this here for learning purposes.
```

### **Convention**

- Uppercase
    
- Words separated by `/` (domain/event format)
    

### **Example**

```js
// src/store/counter/actionTypes.js
export const COUNTER/INCREMENT = "COUNTER/INCREMENT"
export const COUNTER/DECREMENT = "COUNTER/DECREMENT"
export const COUNTER/SET = "COUNTER/SET"
```

### **Tip**

- Use the **state-domain/event-name pattern**  
    → `cart/add-item`, `auth/login-success`, `ui/toggle-modal`
    

---

## **2️⃣ Create Action Creators**

[[Action Creator|Action creators]] are functions that return action objects.

```js
// src/store/counter/actions.js
import { COUNTER/INCREMENT, COUNTER/DECREMENT, COUNTER/SET } from "./actionTypes"

export const increment = () => ({ type: COUNTER/INCREMENT })
export const decrement = () => ({ type: COUNTER/DECREMENT })
export const setCounter = (value) => ({
  type: COUNTER/SET,
  payload: value,
})
```

### **Tips**

- Keep them **pure** and **simple**
    
- Keep naming as **verbs** (“increment”, “reset”, “addItem”)
    

---

## **3️⃣ Create the Reducer**

A reducer controls how state changes based on actions.

```js
// src/store/counter/reducer.js
import { COUNTER/INCREMENT, COUNTER/DECREMENT, COUNTER/SET } from "./actionTypes"

const initialState = {
  value: 0
}

export function counterReducer(state = initialState, action) {
  switch (action.type) {
    case COUNTER/INCREMENT:
      return { ...state, value: state.value + 1 }

    case COUNTER/DECREMENT:
      return { ...state, value: state.value - 1 }

    case COUNTER/SET:
      return { ...state, value: action.payload }

    default:
      return state
  }
}
```

### **Tips**

- Reducers **must** be pure (no API calls, no side effects)
    
- Always return **new objects**, never mutate  
    → `return {...state}` not `state.value = x`
    

---

## **4️⃣ Combine Reducers (If More Than One)**

```js
// src/store/rootReducer.js
import { combineReducers } from "redux"
import { counterReducer } from "./counter/reducer"
import { userReducer } from "./user/reducer"

export const rootReducer = combineReducers({
  counter: counterReducer,
  user: userReducer
})
```

---

## **5️⃣ Create the Store**

```js
// src/store/store.js
import { createStore } from "redux"
import { rootReducer } from "./rootReducer"

export const store = createStore(rootReducer, composeWithDevTools(applyMiddleware(thunk)));
```

---

## **6️⃣ Wrap Your App With `<Provider>`**

```ad-note
This is when we need to use the react-redux package.
```

```jsx
// src/index.js
import React from "react"
import ReactDOM from "react-dom/client"
import { Provider } from "react-redux"
import { store } from "./store/store"
import App from "./App"

ReactDOM.createRoot(document.getElementById("root")).render(
  <Provider store={store}>
    <App />
  </Provider>
)
```

---

## **7️⃣ Use State With `useSelector`**

Using useSelector will create a subscription to the store.

```jsx
import { useSelector } from "react-redux"

export default function CounterValue() {
  const value = useSelector(state => state.counter.value)
  return <p>Count: {value}</p>
}
```

### **Tips**

- Always select the smallest part of the state needed
    
- Avoid selecting large objects (causes unnecessary re-renders)
    

---

## **8️⃣ Dispatch Actions With `useDispatch`**

```jsx
import { useDispatch } from "react-redux"
import { increment, decrement } from "../store/counter/actions"

export default function CounterButtons() {
  const dispatch = useDispatch()

  return (
    <>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </>
  )
}
```

---

# 🎯 **Best Tips and Conventions**

## **1️⃣ Folder Structure**

Use clear separation by domain (recommended):

```
src/
  store/
    counter/
      actionTypes.js
      actions.js
      reducer.js
    user/
      ...
    store.js
    rootReducer.js
```

Or "ducks" style (all in one file per slice) if you want simplicity.

---

## **2️⃣ Action Type Naming**

Use this pattern:

```
featureName/eventName
```

Examples:

- `cart/addItem`
    
- `auth/loginSuccess`
    
- `ui/toggleSidebar`
    

### Why?

- Prevents collisions
    
- Provides context
    
- Easier debugging in Redux DevTools
    

---

## **3️⃣ Action Creator Naming**

Use verbs:

- `addTodo`
    
- `deleteUser`
    
- `toggleTheme`
    
- `resetCart`
    

---

## **4️⃣ Keep Reducers Pure**

Do NOT:

- fetch API inside reducers
    
- mutate arrays/objects
    
- generate random values
    

Reducers should:

- read state
    
- return new state
    

Nothing more.

---

## **5️⃣ Don’t Store Non-Serializable Values**

Avoid in Redux state:

- DOM nodes
    
- class instances
    
- Promises
    
- functions
    

Redux state must remain **serializable** for debugging + devtools.

---

## **6️⃣ Only Use Redux for Cross-App or Multi-Level State**

Do NOT use Redux for:

- local UI toggles
    
- component-only state (use useState)
    

Use Redux when:

- Multiple components need the same state
    
- You need global state
    
- App is large and has complex state logic
    

---

## **7️⃣ Use Redux DevTools**

Always enable it by default.

---

# 🎉 Summary (Very Short)

Redux (classic) flow:  
**action types → actions → reducer → store → provider → useSelector/useDispatch**

Conventions:

- Action type: `"domain/event-name"`
    
- Reducer: pure, return new objects
    
- Structure by domain
    
- Use verbs for action creators
    
- Keep state serializable
    