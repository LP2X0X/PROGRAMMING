---
tags: 
 - react
 - middleware
 - term
---

![[Pasted image 20251115114738.png|center|700]]

```ad-tip
We shouldn’t fetch data directly inside a component and then dispatch it to the store because:

1. We want to keep components clean and focused on UI, not data fetching.
    
2. We want all critical data-fetching logic centralized in one place, rather than scattered throughout the application.
```

## **1. What is Redux Middleware?**

In Redux, **middleware** is a layer between **dispatching an action** and **reaching the reducer**.

Think of it as a **pipeline** where every dispatched action goes through before it hits the reducer.

- **Purpose:** Extend Redux’s behavior without modifying the store.
    
- **Common uses:**
    
    - Logging actions
        
    - Async API calls
        
    - Handling errors
        
    - Analytics tracking
        
    - Routing actions
        

---

## **2. How Middleware Works**

Redux middleware is a function with this **signature**:

```js
const middleware = storeAPI => next => action => {
  // Your code here
};
```

- **storeAPI**: An object with `{ dispatch, getState }`
    
- **next**: Function to pass the action to the next middleware or the reducer
    
- **action**: The Redux action being dispatched
    

**Flow:**

```
dispatch(action) -> middleware1 -> middleware2 -> reducer
```

---

### **Example: Logger Middleware**

```js
const logger = (storeAPI) => (next) => (action) => {
  console.log('Dispatching:', action);
  const result = next(action); // Pass action to next middleware/reducer
  console.log('Next state:', storeAPI.getState());
  return result;
};
```

**How it works:**

1. `dispatch(action)` triggers the logger middleware.
    
2. Middleware can **inspect or modify** the action.
    
3. Calls `next(action)` to continue the chain.
    
4. Can also do things **after the action is processed**, like logging state.
    

---

## **3. Using Middleware in Redux**

### **Classic Redux**

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers';

const store = createStore(
  rootReducer,
  applyMiddleware(logger, thunk) // Multiple middleware
);
```

- `applyMiddleware` chains middleware together.
    
- Order matters: actions flow **in the order they are listed**.
    

---

### **Redux Toolkit (modern)**

```js
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './rootReducer';
import logger from './logger';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger)
});
```

- RTK provides default middleware (e.g., `redux-thunk`) automatically.
    
- You can **add custom middleware** with `.concat()`.
    

---

## **4. Common Middleware Patterns**

### **1. Logging**

```js
const logger = store => next => action => {
  console.log(action.type, action.payload);
  return next(action);
};
```

### **2. Async with Thunks**

```js
const thunkMiddleware = store => next => action => {
  if (typeof action === 'function') {
    return action(store.dispatch, store.getState);
  }
  return next(action);
};
```

- If action is a function, call it with `dispatch` and `getState`.
    
- Otherwise, pass action to reducer.
    

### **3. Error Handling**

```js
const errorMiddleware = store => next => action => {
  try {
    return next(action);
  } catch (err) {
    console.error('Caught an error:', err);
  }
};
```

---

## **5. Key Points About Middleware**

1. **Middleware does not replace reducers.** It only **intercepts actions**.
    
2. **Middleware can modify, delay, or cancel actions.**
    
3. **Order matters.** For example, logging middleware should usually come **last**, so you log the final action.
    
4. **Middleware is composable.** You can chain multiple middleware.
    
5. **Common modern middleware:**
    
    - `redux-thunk`: for async actions
        
    - `redux-saga`: for complex side-effects
        
    - `redux-logger`: for debugging
        

---

## **6. Practical Example (Async API call)**

```js
// Action creator (Thunk)
export const fetchUsers = () => async (dispatch) => {
  dispatch({ type: 'users/fetchStart' });
  try {
    const res = await fetch('https://jsonplaceholder.typicode.com/users');
    const data = await res.json();
    dispatch({ type: 'users/fetchSuccess', payload: data });
  } catch (err) {
    dispatch({ type: 'users/fetchError', payload: err });
  }
};
```

- `redux-thunk` middleware allows **returning a function** instead of an object.
    
- Middleware handles it and passes `dispatch` to the function.
    

---

### **TL;DR Notes**

- Middleware = a function pipeline between dispatch and reducer.
    
- Purpose: logging, async actions, error handling, analytics.
    
- Syntax: `store => next => action => {}`
    
- Call `next(action)` to continue the chain.
    
- Works in both classic Redux (`applyMiddleware`) and Redux Toolkit (`configureStore`).
    