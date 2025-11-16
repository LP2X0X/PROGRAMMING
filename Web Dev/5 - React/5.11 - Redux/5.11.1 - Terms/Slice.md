---
tags: 
 - react
 - redux
 - term
---

# 🧩 **Redux Slice — Detailed Note**

A **slice** in Redux Toolkit (RTK) is a self-contained module that groups **state**, **actions**, and **reducers** for a single feature of your app. It simplifies Redux by removing boilerplate and giving you predictable structure.

---

## 🔹 **1. What a slice contains**

Each slice bundles three things:

1. **A piece of state** (e.g., `todos`, `auth`, `cart`) and its intial value
    
2. **Reducers** that update that state
    
3. **Automatically generated action creators + action types**
    

All defined in one place.

---

## 🔹 **2. How to create a slice**

```js
import { createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",         // namespace
  initialState: { value: 0 },
  reducers: {
    increment(state) {
      state.value++;       // allowed because of Immer
    },
    add(state, action) {
      state.value += action.payload;
    },
  },
});

export const { increment, add } = counterSlice.actions; // auto-generated
export default counterSlice.reducer;
```

---

## 🔹 **3. Why slices are useful**

### ✅ **1. No more manual action types**

RTK generates action types like:

```
counter/increment
counter/add
```

### ✅ **2. No need for switch statements**

Reducers are defined inline as functions — much cleaner.

### ✅ **3. Mutating state is allowed**

Because RTK uses **Immer**, you can write:

```js
state.value++
```

But Immer safely produces a new state immutably.

### ✅ **4. Each slice is modular**

You can keep logic organized by feature:

- `authSlice`
    
- `cartSlice`
    
- `todoSlice`
    
- `uiSlice`
    

Perfect for scaling large applications.

---

## 🔹 **4. Using a slice in the store**

```js
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./counterSlice";

const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```

Now your state becomes:

```js
state.counter.value
```

---

## 🔹 **5. Using slices in React components**

### **Dispatch actions**

```js
import { useDispatch } from "react-redux";
import { increment } from "./counterSlice";

const dispatch = useDispatch();
dispatch(increment());
```

### **Select state**

```js
import { useSelector } from "react-redux";

const value = useSelector(state => state.counter.value);
```

---

## 🔹 **6. Slices can include async logic (extraReducers)**

Used with `createAsyncThunk`:

```js
extraReducers: builder => {
  builder.addCase(fetchUser.fulfilled, (state, action) => {
    state.user = action.payload;
  });
}
```

---

## 🔹 **7. Slice naming conventions**

- Slice `name` = feature domain
    
- Reducer keys = event names
    

Example:

```js
name: "cart",
reducers: {
  addItem,
  removeItem,
}
```

RTK auto-generates:

```
cart/addItem
cart/removeItem
```

---

# 📝 **In summary**

A **Redux Slice**:

- Organizes state + actions + reducers into a single module
    
- Generates action creators and type strings automatically
    
- Lets you write “mutating” code safely
    
- Makes Redux dramatically simpler and more compact
    
- Is the main building block of Redux Toolkit apps
    