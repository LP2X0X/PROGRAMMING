---
tags: 
 - react
 - redux
 - toolkit
---

`configureStore()` is the main function from **Redux Toolkit (RTK)** used to create your Redux store.

It is the **recommended** way to set up Redux because it:

- automatically applies good defaults
    
- reduces boilerplate
    
- sets up Redux DevTools
    
- adds built-in middleware (like `redux-thunk`)
    

---

## ✅ **What `configureStore()` Does**

When you call it, `configureStore()`:

1. **Combines your reducers**
    
2. **Adds useful default middleware**
    
    - `redux-thunk`
        
    - DevTools middleware
        
    - checks for common mistakes (immutability, serializable actions)
        
3. **Enables Redux DevTools** automatically
    
4. **Creates the store object** (same as the old `createStore`)
    

All with minimal config.

---

# 📦 Basic Example

```js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```

This creates a store with:

- a `counter` slice reducer
    
- default middleware
    
- devtools enabled
    

---

# 📌 Options You Can Pass

### ⭐ **1. `reducer` (required)**

Object of reducers or a single reducer.

```js
configureStore({
  reducer: {
    user: userReducer,
    cart: cartReducer,
  }
});
```

````ad-note

Reducers are passed as an object because the **object keys become the state slices**, and the **values are the reducers that manage those slices**.  
So:

```js
reducer: {
  user: userReducer,
  cart: cartReducer
}
```

creates a store state shaped like:

```js
{ user: ..., cart: ... }
```

Redux uses the keys (`user`, `cart`) to know **where each reducer’s data lives** and automatically combines them internally.

````

---

### ⭐ **2. `middleware`**

You can extend or replace the default middleware.

Extend:

```js
configureStore({
  reducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(loggerMiddleware),
});
```

Replace:

```js
configureStore({
  reducer,
  middleware: () => [loggerMiddleware],
});
```

---

### ⭐ **3. `devTools`**

Enable/disable Redux DevTools:

```js
configureStore({
  reducer,
  devTools: process.env.NODE_ENV !== "production",
});
```

---

### ⭐ **4. `preloadedState`**

Initial state (useful for SSR):

```js
configureStore({
  reducer,
  preloadedState: {
    user: { name: "Long" }
  }
});
```

---

### ⭐ **5. `enhancers`**

Extend store behavior:

```js
configureStore({
  reducer,
  enhancers: (defaultEnhancers) => defaultEnhancers,
});
```

---

# 📚 What You Get Back

```js
const store = configureStore({ ... });
```

You get an object with:

- `store.dispatch`
    
- `store.getState`
    
- `store.subscribe`
    
- `store.replaceReducer`
    
- a typed `RootState` and `AppDispatch` (TypeScript)
    

---

# 🧠 Why Use `configureStore`?

Because it removes **all the painful config** you had to do manually with vanilla Redux.

It automatically:

- turns on DevTools
    
- includes middleware
    
- enforces good practices
    
- works perfectly with RTK slices
    
- reduces boilerplate by 80%
    

It’s simply **the correct way to configure a Redux store in 2025**.
