---
tags: 
 - react
 - redux
 - term
---

In **Redux**, the **store** is the central place that holds the **state** of your entire application. Think of it as a single source of truth for all the data your app needs. It is more than just a simple object—it also provides methods to read state, update state, and subscribe to changes.

---

### **1. Core Responsibilities of the Store**

1. **Holds the state**  
    The store keeps the current state of the app. In Redux, state is always **immutable**, so updates always produce a new state object.
    
2. **Allows access to state**  
    Using the `getState()` method, you can read the current state.
    
    ```js
    const state = store.getState();
    console.log(state);
    ```
    
3. **Allows state updates via dispatching actions**  
    You **never modify the state directly**. Instead, you **dispatch actions** to the store, which are then handled by reducers to produce a new state.
    
    ```js
    store.dispatch({ type: 'increment' });
    ```
    
4. **Registers listeners via subscription**  
    You can subscribe to the store to listen for state changes:
    
    ```js
    const unsubscribe = store.subscribe(() => {
      console.log('State changed:', store.getState());
    });
    ```
    

---

### **2. How the Store is Created**

You create a Redux store using `createStore` (or `configureStore` in Redux Toolkit):

```js
import { createStore } from 'redux';
import rootReducer from './reducers';

const store = createStore(rootReducer);
```

- `rootReducer` is a function that receives the current state and an action, and returns the new state.
    

---

### **3. How it Fits in a React App**

- The **store** is usually provided to your React app via the `<Provider>` from `react-redux`.
    
- Components can then access the state with `useSelector` and dispatch actions with `useDispatch`.
    

```js
import { Provider } from 'react-redux';
import store from './store';
import App from './App';

<Provider store={store}>
  <App />
</Provider>
```

---

✅ **Summary:** The **Redux store** is like a state manager. It holds your app’s state, lets you read it, lets you update it only through actions, and lets you react to changes via subscriptions.