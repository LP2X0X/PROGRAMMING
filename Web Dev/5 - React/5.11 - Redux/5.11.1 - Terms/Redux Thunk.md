---
tags: 
 - react
 - redux
 - middleware
---

![[Pasted image 20251116095724.png|center|700]]

A _side effect_ is **anything that happens outside the pure reducer**, such as:

- fetching data from a server
    
- updating localStorage
    
- logging
    
- timing (setTimeout)
    
- reading current state before deciding
    
- dispatching multiple actions over time
    

Reducers must stay pure:

```
(state, action) → newState  // no async, no fetch, no mutation
```

So you need **middleware** to handle side effects.

---

# 🎯 Redux Thunk’s job

> **Let you run side effects, then dispatch actions afterward.**

Example side effect: _fetch from API_

```js
export const loadPosts = () => async (dispatch) => {
  dispatch(postsLoading());
  
  const res = await fetch('/posts');
  const data = await res.json();

  dispatch(postsLoaded(data));
};
```

Everything inside the thunk is a **side effect**:

- network request
    
- async/await
    
- dispatching multiple actions
    
- maybe reading state
    


> **Redux Thunk is for side effects (especially async ones like fetching).**

---

# ⭐ 1. What Problem Redux Thunk Solves

Redux reducers must be:

- **pure functions**
    
- **synchronous**
    
- **return the next state immediately**
    

So reducers **cannot**:

- wait for `fetch()` results
    
- call timeouts
    
- do async logic
    
- perform side effects (logging, localStorage, server calls)
    

Example:  
You want to fetch user data in Redux:

❌ **Without thunk this is illegal**:

```js
dispatch({ type: "user/fetch" });  // Cannot fetch in a reducer!
```

Redux won’t allow code like:

```js
function userReducer(state, action) {
  if (action.type === "user/fetch") {
    const data = await fetch(...);  // ❌ Not allowed inside reducer
  }
}
```

```ad-summary
The Thunk allows Redux to wait before dispatching the fetch data (in action) to the store.
```

---

# ⭐ 2. What Redux Thunk Actually Does

Redux Thunk allows you to dispatch a **function**, not just an object.

Normally, you dispatch:

```js
dispatch({ type: "SOME_ACTION", payload: ... })
```

With thunk, you can dispatch:

```js
dispatch(function (dispatch, getState) {
  // async logic here
});
```

The callback function is called a **thunk**.

### Why?

Because a “thunk” is a function that delays work until it is executed.

```ad-note
`getState` lets your thunk **read the current Redux state** so you can _decide what to do before dispatching anything_.

Example:

- avoid refetching if data is already loaded
    
- check authentication before calling an API
    
- read a value needed to build the request
    

So: **`getState()` = access store state inside async logic.**
```

---

# ⭐ 3. Example (basic thunk)

### Without thunk — illegal because async in reducer:

```js
dispatch({ type: "user/fetch" });
```

### With thunk:

```js
function fetchUser() {
  return async function (dispatch, getState) {
    dispatch({ type: "user/loading" });

    const data = await fetch("/api/user").then(res => res.json());

    dispatch({ type: "user/loaded", payload: data });
  };
}

dispatch(fetchUser());
```

Key idea:

- `fetchUser()` returns a function
    
- thunk middleware sees the function → executes it instead of passing to reducer
    
- thunk gives it `dispatch` and `getState`
    
- your async logic runs here
    
- final actions are normal object actions (reducers can handle them)
    

---

# ⭐ 4. How to install & enable Redux Thunk

```bash
npm install redux-thunk
```

Add middleware:

```js
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import rootReducer from "./rootReducer";

const store = createStore(rootReducer, applyMiddleware(thunk));
```

Now your Redux store supports thunks.

---

# ⭐ 5. Full Example — Step-by-step

### Step 1: Reducer

```js
const initialState = {
  balance: 0,
  loan: 0,
  loanPurpose: "",
  isLoading: false,
};

function accountReducer(state = initialStateAccount, action) {
  switch (action.type) {
    case "account/deposit":
      return { ...state, balance: state.balance + action.payload };
    case "account/withdraw":
      if (state.balance < action.payload) return state;
      return { ...state, balance: state.balance - action.payload };
    case "account/requestLoan":
      if (state.loan > 0) return state;
      return {
        ...state,
        balance: state.balance + action.payload.amount,
        loan: action.payload.amount,
        loanPurpose: action.payload.amount,
      };
    case "account/payLoan":
      return {
        ...state,
        balance: state.balance - state.loan,
        loan: 0,
        loanPurpose: "",
      };
    default:
      return state;
  }
}
```

---

### Step 2: Thunk Action Creator

```js
export function deposit(amount, currency) {
  // In case we don't need side effect
  if (currency === "USD") return { type: "account/deposit", payload: amount };

  // If we do
  return async function (dispatch, getState) {
    dispatch({ type: "account/convertingCurrency" });

    const res = await fetch(
      `https://api.frankfurter.app/latest?amount=${amount}&from=${currency}&to=USD`
    );
    const data = await res.json();
    const converted = data.rates.USD;

    dispatch({ type: "account/deposit", payload: converted });
  };
}
```

```ad-note
We pass `dispatch` into a thunk because the dispatch that handled the original action creator cannot directly handle asynchronous logic. The thunk receives `dispatch` so that, after completing its async work, it can manually dispatch the real actions at the appropriate time.

**Thunk middleware steps:**
1. `dispatch` sees the value is **a function**
    
2. Instead of sending it to reducers, thunk **executes the function**
    
3. That function receives `(dispatch, getState)` (The same outer dispatch function)
    
4. Inside the thunk, you can perform the async work and then dispatch the appropriate actions only after that work has completed

---

> **Redux dispatch expects a plain object action. But when you dispatch a function (a thunk), the thunk middleware intercepts it and calls that function, passing in `dispatch` and `getState`. The thunk then performs its async logic and manually calls `dispatch` again when it’s ready.**

This clarifies a few important points:

1. Redux itself cannot handle async

Only middleware can understand things that aren’t plain objects.

2. When you dispatch a thunk, Redux never actually sees it

The thunk middleware catches it first.

3. The middleware gives the thunk _its own copy_ of dispatch

This lets the thunk:

- start async work
    
- wait
    
- then call `dispatch(...)` again whenever it finishes
    

4. The “real” action is dispatched later

The thunk is not the real action — just a wrapper around async.

> **Thunk = async function that receives dispatch so it can run async stuff and dispatch real actions afterwards.**

```
---

### Step 3: Using it in a component

```jsx
function handleDeposit() {
	if (!depositAmount) return;
	
	dispatch(deposit(depositAmount, currency));
	setDepositAmount("");
	setCurrency("USD");
}
```

---

# ⭐ 6. WHY this works — behind the scenes (mechanism)

**Normal Redux dispatch:**

```js
store.dispatch(action);
```

- Only accepts objects
    
- Goes directly to reducer
    

**With thunk middleware added:**

Middleware wraps dispatch:

```js
function thunkMiddleware({ dispatch, getState }) {
  return function (next) {
    return function (action) {
      if (typeof action === "function") {
        // special case for thunks
        return action(dispatch, getState);
      }

      // otherwise, pass to reducer
      return next(action);
    };
  };
}
```

This means:

### ✔ If you dispatch an object → normal behavior

### ✔ If you dispatch a function → thunk runs it instead of reducer

Thunk _intercepts_ the dispatched value and makes the decision.

---

# ⭐ 7. Why thunk is powerful

Because you can do literally anything inside a thunk:

### ✔ async/await

### ✔ fetch API

### ✔ setTimeout

### ✔ logging

### ✔ access current store state

### ✔ dispatch multiple actions

### ✔ cancel logic based on `getState()`

### ✔ side effects (analytics, localStorage)

Reducers stay pure and sync, while thunks handle the messy work.

---

# ⭐ 8. Best Tips / Conventions

### **1. Always dispatch loading + success actions**

```js
dispatch({ type: "posts/loading" });
dispatch({ type: "posts/loaded", payload: data });
```

### **2. Put all async logic inside thunk — not in components**

Keep components clean.

### **3. Do not mutate state inside thunk**

Only reducers should modify state.

### **4. Thunks can return promises**

So components can:

```js
await dispatch(fetchUser());
```

### **5. Thunks can read current state**

Useful for:

```js
const alreadyLoaded = getState().user.loaded;
```

---

# ⭐ 9. When to use thunk

Use thunk when you need:

✔ API calls  
✔ async flow  
✔ sequential actions  
✔ conditional dispatch  
✔ timers  
✔ combining multiple actions  
✔ reading state before dispatching

---

# ⭐ 10. When NOT to use thunk

- If your async logic is complex → use RTK Query or Sagas
    
- If you need websockets → sagas or observables are better
    
- If you want simpler API → use Redux Toolkit (`createAsyncThunk`)
    