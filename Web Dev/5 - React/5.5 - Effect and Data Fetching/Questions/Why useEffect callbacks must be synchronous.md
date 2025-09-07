---
tags: 
 - react
 - question
---

## 🔹 First: Why `useEffect` callbacks must be synchronous

React’s design expects the effect function to:

1. **Run synchronously after render** so React knows exactly:
    
    - if it needs to clean up an old effect
        
    - when to run the new effect
        
2. **Return either nothing or a cleanup function**.
    
    - If you made the effect itself `async`, it would return a **Promise** instead of a cleanup function.
        
    - React wouldn’t know when to clean up → risk of memory leaks and race conditions.
        

That’s why React **forbids** making `useEffect` directly `async`.

---

## 🔹 Where race conditions come from

Imagine you fetch data in `useEffect`:

```jsx
useEffect(() => {
  async function fetchData() {
    const res = await fetch("/api/data");
    const json = await res.json();
    setData(json);
  }

  fetchData();
}, []);
```

Problem:

- If the component re-renders quickly (say props change) → **multiple fetches** may be in-flight.
    
- The “slower” request might finish last, but overwrite data from the newer request → **stale data bug**.
    

That’s the **race condition**.

---

## 🔹 Why putting async _inside_ helps

By writing the async logic **inside** the effect, you can:

- Define an **abort flag** (or use `AbortController`) that checks if the component is still mounted or the effect is still “valid”.
    
- Cleanly cancel or ignore outdated promises.
    

Example fix:

```jsx
useEffect(() => {
  let isCancelled = false;

  async function fetchData() {
    const res = await fetch("/api/data");
    const json = await res.json();

    if (!isCancelled) {
      setData(json); // only update if still valid
    }
  }

  fetchData();

  return () => {
    isCancelled = true; // cleanup prevents race conditions
  };
}, []);
```

Now:

- If the component unmounts or dependencies change, `isCancelled` flips.
    
- Old async tasks **won’t update state** → race condition avoided.
    

---

## ✅ Summary

- `useEffect` must be synchronous → React expects a cleanup function, not a Promise.
    
- Async in effects can cause **race conditions** if multiple async tasks overlap.
    
- Putting async inside the effect lets you use cleanup/flags to prevent stale updates.
    
