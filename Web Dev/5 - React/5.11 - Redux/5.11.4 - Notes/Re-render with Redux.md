---
tags: 
 - react
 - redux
 - re-render
---

React components **only re-render when the specific slice of Redux state they _read_** changes — not when anything else in the store changes.

---

# ✅ **When a Component Re-renders with Redux**

A component re-renders **only when:**

### **1️⃣ The selector function returns a NEW value**

Example:

```js
const customer = useSelector(state => state.customer)
```

This component re-renders **only if `state.customer` is different** from the last time.

React-Redux does:

1. Call your selector with the new state
    
2. Compare the returned value to the previous one
    
3. If changed → trigger re-render
    
4. If not changed → skip re-render
    

The comparison is done with **shallow equality** (`===`).

---

# ❌ **When the Component Does NOT Re-render**

It **does NOT re-render** if some other part of the store changes:

Example:

```js
const customer = useSelector(s => s.customer)
```

What if `state.cart` changes?  
→ **No re-render**  
What if `state.user` changes?  
→ **No re-render**

Even if the store is huge — only the selected slice matters.

---

# 🧠 **Key Rule**

> React-Redux re-renders a component only if the value returned from `useSelector` changed by reference (`!==`).

---

# 🔍 **Examples**

## 🔹 1. If the reducer returns a new object → component re-renders

```js
return { ...state, name: "Alice" } // new object
```

New reference → re-render.

---

## 🔹 2. If the reducer mutates the object but returns the same reference → component does NOT re-render

```js
state.name = "Alice";
return state; // ❌ bad — same reference
```

React-Redux thinks nothing changed → **no re-render**  
(This is why reducers must be pure and immutable.)

---

## 🔹 3. Selecting a primitive (string, number)

```js
useSelector(s => s.count)
```

Re-renders when:

- count changes from 1 → 2
    
- comparison is by value
    

---

## 🔹 4. Selecting a dynamic or new object each time

Bad example:

```js
useSelector(s => ({ total: s.cart.total }))
```

This ALWAYS returns a **new object**, so the component re-renders **every time**, even if `cart.total` didn’t change.

---

# 🟢 **Best Practice**

Always select the smallest stable value you need:

✔ Good:

```js
useSelector(s => s.cart.total)
```

❌ Bad:

```js
useSelector(s => s.cart)
```

(Re-renders for any cart change)

❌ Very Bad:

```js
useSelector(s => ({ total: s.cart.total }))
```

(Creates a new object every time → always re-render)

---

# 🎯 Final Summary

### A component using Redux re-renders ONLY when:

**the value your selector returns changes (=== comparison).**

✔ The selected state changed by reference  
✔ The reducer created a new object/array  
✔ The primitive value changed

### A component does NOT re-render when:

❌ other parts of the store change  
❌ reducer mutates the state instead of returning new one  
❌ selector returns the same reference as last time

</br>

----

</br>

# Example

```js
const initialStateCustomer = { fullname: "", nationalID: "", createdAt: "", };
```

# ✅ Component Example

```js
const fullname = useSelector(state => state.customer.fullname);
```

# 🔍 What happens?

1. Redux store updates because an action is dispatched
    
2. React-Redux runs your selector:
    
    ```js
    state => state.customer.fullname
    ```
    
3. It compares the **new returned value** with the **previous returned value** using `===`
    

---

# 🧠 The re-render rule

### ✔ Re-render

If the new `fullname` value is different:

|previous|new|
|---|---|
|`"Alice"`|`"Bob"`|

→ Re-render

---

### ❌ No re-render

If **fullname stays the same**, even if other fields change:

Example action:

```js
dispatch(createCustomer("Alice", "12345"));
```

This changes:

- fullname → "Alice"
    
- nationalID → "12345"
    
- createdAt → "2024-01-01T00:00Z"
    

But if your component selects only fullname and fullname stays equal to the previous value, React-Redux compares:

```js
"Alice" === "Alice"  // true → no re-render
```

Even though nationalID and createdAt changed → **your component does NOT re-render**.

---

# 🧪 Example Scenarios

## Scenario 1: `updateName`

```js
dispatch(updateName("John"));
```

Your selector picks `"John"` → changed → **component re-renders**

---

## Scenario 2: Calling createCustomer with SAME fullname

```js
dispatch(createCustomer("John", "8888"));
```

If fullname was already `"John"` before this update:

- fullname stays `"John"`
    
- nationalID changes
    
- createdAt changes
    
- → but selector returns `"John"`
    

`"John" === "John"` → **no re-render**

---

## Scenario 3: CreatedAt changes, fullname does not

```js
dispatch({ type: "customer/updateCreatedAt" })
```

If this action changes only `createdAt`, and your selector is:

```js
useSelector(s => s.customer.fullname)
```

Result:

- createdAt changed (irrelevant)
    
- fullname same
    
- selector returns the same value → **no re-render**
    

---

# 🎯 Bottom Line

### ✔ If your component selects _only_ fullname →

**it re-renders ONLY when fullname changes.**

### ❌ It does NOT re-render when:

- nationalID changes
    
- createdAt changes
    
- other reducers' state changes
    
- any unrelated update happens in the app
    