---
tags: 
 - react
 - redux
 - function
 - selector
---

## 1. What a Selector Is

A **selector** is simply a _pure function_ that takes the Redux **state tree** and returns a **derived value** from it.

**Signature:**

```js
(state) => state.someSlice.value
```

Selectors:

- Encapsulate how state is structured.
    
- Prevent components from knowing internal slice layout.
    
- Improve testability.
    
- Allow memoization for derived values.
    

---

## 2. Basic Selector (Non-memoized)

If you have a slice called `counter`:

```js
// store.js
{
  counter: {
    value: 5
  }
}
```

Selector:

```js
export const selectCounterValue = (state) => state.counter.value;
```

Usage in React:

```js
const value = useSelector(selectCounterValue);
```

---

## 3. Why Use Selectors

1. **Encapsulation**  
    Components do not depend on the exact state structure.
    
2. **Better reuse**  
    Multiple components can use the same selector.
    
3. **Easy testing**  
    You can test selectors independently without rendering components.
    
4. **Memoization (via Reselect)**  
    Avoid unnecessary recalculations when computing derived data.
    

---

## 4. Memoized Selectors with `createSelector`

When you need to compute derived data (filters, totals, expensive calculations), use `createSelector` from Reselect (built into Redux Toolkit):

```js
import { createSelector } from '@reduxjs/toolkit';

const selectItems = (state) => state.cart.items;

export const selectTotalPrice = createSelector(
  [selectItems],
  (items) => items.reduce((sum, item) => sum + item.price, 0)
);
```

Behavior:

- `selectTotalPrice` only recomputes when `state.cart.items` changes.
    
- `useSelector(selectTotalPrice)` receives stable results and avoids unnecessary re-renders.
    

---

## 5. Selector Factories (Selectors That Receive Parameters)

Use this pattern when you need a selector that depends on dynamic arguments.

```js
export const makeSelectItemById = (id) =>
  createSelector(
    [(state) => state.cart.items],
    (items) => items.find((x) => x.id === id)
  );
```

Usage:

```js
const item = useSelector(makeSelectItemById(productId));
```

Each call creates a new memoized selector instance.

---

## 6. Selectors with Redux Toolkit Slice

You may define selectors inside or next to the slice:

```js
// counterSlice.js
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value++ }
  }
});

export const selectCount = (state) => state.counter.value;
```

---

## 7. Best Practices

- Export selectors from each slice file as part of its public API.
    
- Prefer selectors over directly accessing state in components.
    
- Use memoization only for derived or expensive computations.
    
- Keep selectors pure. Do not mutate state or use side effects.
    
