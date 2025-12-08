---
tags: 
 - react
 - useEffect
 - note
---

- For example, in a context provider component, we have this function:
```js  
async function getCity(id) {
    if (Number(id) === currentCity) {
      return;
    }

    try {
      dispatch({ type: "loading" });
      const res = await fetch(`${BASE_URL}/cities/${id}`);
      if (!res.ok) {
        throw new Error("There is something wrong when fetching city data!");
      }
      const data = await res.json();
      dispatch({ type: "city/loaded", payload: data });
    } catch (err) {
      dispatch({ type: "rejected", payload: err.message });
    }
  }
```

- In some components, we access a function from the context and use it within an effect:
```js
const { getCity, currentCity, isLoading } = useCities();

useEffect(function () {
  getCity(id);
}, [id, getCity]);
```

- Now, here's the problem: getCity update the state inside the context each time that it is executed which in turn cause the context provider to re-render. Therefore, a new getCity function is created. Since a new getCity is created and also it is in the dependency array of the above useEffect, getCity will be called again. In conclusion, it will cause an **infinite loop**.

---

React warns:

**“Calling setState synchronously within an effect can trigger cascading renders.”**

This happens when your `useEffect` runs **synchronously calls a function that calls `setState` immediately**, causing:

1. `useEffect` runs
    
2. Inside it, you call a function immediately
    
3. That function calls `setState`
    
4. Component re-renders
    
5. **React immediately runs the effect again** after the re-render
    
6. This repeats → Cascading renders (infinite or heavy loop)
    

React expects `useEffect` to:

- Subscribe to external systems, OR
    
- Sync external systems with React state
    

NOT to call `setState` immediately inside the effect body.

### 2. Example of the problem

#### Problematic code (causes the warning)

```jsx
useEffect(() => {
  updateTimeLeft(); // updateTimeLeft() calls setState() immediately
}, []);
```

Where:

```jsx
function updateTimeLeft() {
  setMinuteLeft(5);
}
```

This produces:

- effect runs
    
- updateTimeLeft() runs → setState()
    
- state updates → re-render
    
- effect runs again
    
- updateTimeLeft() runs again → setState()
    
- infinite loop
    

React warns because this pattern **almost always indicates a logical bug**.

### 3. Correct fix (run state changes in async callback, like interval)

Instead of calling state updates _synchronously_ inside the effect, run them inside a **timer, subscription, or async job**.

#### Correct version

```jsx
useEffect(() => {
  function updateTimeLeft() {
    const timeLeft =
      new Date(estimatedDelivery).getTime() - new Date().getTime();

    setMinuteLeft(Math.floor(timeLeft / (60 * 1000)));
  }

  // Run immediately ONCE
  updateTimeLeft();

  // Run on interval afterwards
  const id = setInterval(updateTimeLeft, 1000 * 60);

  return () => clearInterval(id);
}, [estimatedDelivery]);
```

#### Why this is correct

- You call `updateTimeLeft()` once when the effect runs
    
- Further calls happen **inside the interval callback**, not synchronously inside the effect
    
- React re-renders, but does **not** restart the interval immediately
    
- No recursion, no cascading renders
    

### 4. Another valid pattern (if you must compute once)

If you only want to compute state once, do it _during initial render_, not inside the effect:

```jsx
const [minuteLeft, setMinuteLeft] = useState(() => {
  const timeLeft =
    new Date(estimatedDelivery).getTime() - new Date().getTime();
  return Math.floor(timeLeft / (60 * 1000));
});
```

This avoids effects entirely and is recommended when you only need one computation.

#### Summary

React warns because:

- **Calling setState inside the effect body triggers a re-render**
    
- The re-render re-runs the effect
    
- That re-runs the state update again
    
- → Infinite render loop or cascading renders
    

Use **intervals, async callbacks, or lazy initial state** instead.
