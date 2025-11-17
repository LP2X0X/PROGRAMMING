---
tags: 
 - react
 - term 
 - redux
 - state
 - management
---

Redux is a **state management library** that helps you manage **global application state** in a predictable, centralized way. It is not tied to React, but it is most commonly used with it.

![[Pasted image 20251114195056.png|center|700]]


---

# 1️⃣ The Core Idea of Redux

Redux is built on **one big principle**:

> **All global state lives in a single store, and it can only be updated through pure functions called reducers.**

This gives you:

- Predictability
    
- Traceability
    
- Easier debugging
    
- A clear mental model for how changes happen
    

---

# 2️⃣ The Three Core Principles of Redux

### **1. Single source of truth**

All your global state is stored in **one object** (the store).

### **2. State is read-only**

You cannot mutate state directly.  
To change state, you must **dispatch an action**.

### **3. Changes are made with pure reducers**

A reducer is a pure function:

```js
(prevState, action) => newState
```

No:

- Side effects
    
- DOM manipulation
    
- Randomness
    
- API calls
    

Reducers simply **compute the next state** based on the current state + the action.

---

# 3️⃣ The Building Blocks of Redux

## 🔸 Store

Holds the application state.

```js
const store = createStore(rootReducer, composeWithDevTools(applyMiddleware(thunk)));
```

## 🔸 Actions

Plain JS objects describing _what happened_.

```js
{ type: 'cart/addItem', payload: item }
```

They do **not** describe how the state changes—only what occurred.

```ad-tip
The action type should follow the **"state-domain/event-name"** pattern, meaning it clearly identifies which part of the state it belongs to and describes the specific event, making actions easier to organize, read, and debug.
```

```ad-tip
We usually don't write action manually but used [[Action Creator|action creator]].
```

## 🔸 Dispatch

The method used to send actions to the store.

```js
store.dispatch({ type: 'cart/addItem', payload: item });
```

## 🔸 Reducers

Pure functions that return the new state.

```js
function cartReducer(state = initialStateCart, action) {
  switch (action.type) {
	  case 'cart/addItem':
		  return {...state, items: [...items, action.payload]};
	  default:
		  return state;
  }
}
```

```ad-note
Each reducer must give its `state` parameter a default value equal to its initial state, because Redux will call the reducer on startup, and the reducer must return the initial state in that case.
```

```ad-note
In Redux, the reducer’s default case must always return the existing state. Unlike React’s `useReducer`, you cannot return nothing or `undefined`; Redux requires the reducer to always return a valid state object.
```

In case you have multiple reducers. 

```js
const rootReducer = combineReducers({
  account: accountReducer,
  customer: customerReducer,
});
```

````ad-note
- `combineReducers` takes an **object** where:
    
    - **key** = the **name of the slice in the state tree**
        
    - **value** = the reducer function that manages that slice
        
- In this case:
    
    - `account` → managed by `accountReducer`
        
    - `customer` → managed by `customerReducer`
        

Here's what the state tree looks like:

```js
{
  account: {
    balance: 0,
    loan: 0,
    loanPurpose: '',
    isLoading: false
  },
  customer: {
    fullname: '',
    nationalID: '',
    createdAt: ''
  }
}
```

- So when you want to access state in a component:
    

```js
const balance = useSelector(state => state.account.balance);
const fullname = useSelector(state => state.customer.fullname);
```

- `account` and `customer` **are the keys in the state tree**, exactly as defined in `combineReducers`.
    
````

````ad-note
When you have **only one reducer in your store**:

```js
import { createStore } from 'redux';
import counterReducer from './counterReducer';

const store = createStore(counterReducer);
```

- The **entire state tree** is **whatever `counterReducer` returns**.
    
- There is no nested property, because you didn’t use `combineReducers`.
    

Accessing state in React:

```js
const value = useSelector((state) => state.value);
```

- `state` here = the state returned by your single reducer.
    
- `value` = the property inside that reducer’s state.
    

✔ **Correct, standard convention** for a single reducer store.
````

## 🔸 Selectors

Functions that read state from the store.

```js
const items = useSelector(state => state.cart.items);
```

---

# 4️⃣ How Redux Flows (Unidirectional Data Flow)

```
UI → dispatch(action) → reducer(state, action) → store updates → UI re-renders
```

This predictable flow is why Redux is called a **unidirectional state container**.

![[Pasted image 20251114195846.png|center|700]]
![[Pasted image 20251114195956.png|center|700]]

---

# 5️⃣ Example with Classic Redux

### actions.js

```js
// Action types
export const INCREMENT = 'counter/increment';
export const ADD_BY = 'counter/addBy';

// Action creators
export const increment = () => ({ type: INCREMENT });
export const addBy = (amount) => ({ type: ADD_BY, payload: amount });
```

### reducer.js

```js
const initialState = { value: 0 };

export default function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'counter/increment':
      return { ...state, value: state.value + 1 };
    case 'counter/addBy':
      return { ...state, value: state.value + action.payload };
    default:
      return state;
  }
}
```

### store.js

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import counterReducer from './reducer';

const store = createStore(counterReducer, applyMiddleware(thunk));

export default store;
```

### index.js
```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import store from "./store";
import { Provider } from "react-redux";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

### Component.jsx

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { increment, addBy } from './actions';

function Counter() {
  const value = useSelector((state) => state.value); // Since there is only one reducer -> one state -> use the word state and not store
  const dispatch = useDispatch();

  return (
    <>
      <p>{value}</p>
      <button onClick={() => dispatch(increment())}>+1</button>
      <button onClick={() => dispatch(addBy(5))}>+5</button>
    </>
  );
}

export default Counter;
```

---

# 6️⃣ Async Logic in Classic Redux (with Redux Thunk)

### actions.js

```js
export const FETCH_USERS_PENDING = 'users/fetch_pending';
export const FETCH_USERS_SUCCESS = 'users/fetch_success';
export const FETCH_USERS_ERROR = 'users/fetch_error';

// Thunk action creator
export const fetchUsers = () => {
  return async (dispatch) => {
    dispatch({ type: FETCH_USERS_PENDING });

    try {
      const res = await fetch('/api/users');
      const data = await res.json();
      dispatch({ type: FETCH_USERS_SUCCESS, payload: data });
    } catch (err) {
      dispatch({ type: FETCH_USERS_ERROR, payload: err.message });
    }
  };
};
```

### reducer.js

```js
const initialState = { users: [], loading: false, error: null };

export default function usersReducer(state = initialState, action) {
  switch (action.type) {
    case 'users/fetch_pending':
      return { ...state, loading: true, error: null };
    case 'users/fetch_success':
      return { ...state, loading: false, users: action.payload };
    case 'users/fetch_error':
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}
```

This is the **classic Redux approach**:

- You manually define action types and creators
    
- Use **thunks** to handle async logic
    
- Reducers update state based on dispatched actions
    

---

# 7️⃣ When to Use Redux

Redux is beneficial when your app has:

- Shared state used across many components
    
- Complex state transitions
    
- Debug-heavy logic
    
- Data from multiple sources
    
- Need for predictable state changes
    
- Large app with multiple developers
    

It may be **overkill** for small apps.

![[Pasted image 20251114195221.png|center|700]]

Redux is used less today because most modern apps don’t have much **global UI state**—the type of state Redux was built for. Instead, most state now comes from the **server**, and server-state is much better handled by tools like **React Query, SWR, Apollo, tRPC, and RTK Query**, which offer caching, background refetching, deduping, and server synchronization—features Redux wasn’t designed for. As a result, apps often manage small UI state with React and let server-state libraries handle the rest, reducing the need for Redux.

---

# 8️⃣ Redux vs Context API

|Feature|Redux|Context API|
|---|---|---|
|Purpose|Global state with logic|Passing values down a tree|
|Performance|Highly optimized|Re-renders often|
|DevTools|Excellent|None|
|Async handling|Built-in (thunks)|Manual|
|Best for|Large, complex apps|Small/medium apps|

Context is **not a state management solution**.  
Redux is.

---

# 9️⃣ Summary (Cheat Sheet)

- Redux stores global state in one place
    
- Actions describe _what_ happened
    
- Reducers describe _how_ state changes
    
- Strict unidirectional flow
    
- Redux Toolkit is the modern, recommended way
    
- Use Redux for medium → large apps with complex shared state
    