---
tags: 
 - react
 - note
---

Even though React state updates are **asynchronous and batched**, React still guarantees that **your `setState` calls are processed in the order they were called**.

---

### Breaking it down with your example:

```js
setAvgRating(Number(imdbRating)); 

setAvgRating(avgRating => (avgRating + userRating) / 2);
```

1. **First call**: React schedules an update to replace `avgRating` with `Number(imdbRating)`.
    
2. **Second call**: React schedules another update. But because this one is a **function updater**, React won’t run it right away. Instead, it queues it and when processing updates, it will call it with the **latest state value so far** (which includes the effect of the first update).
    

So even though batching means React might wait until after the event handler finishes to apply updates, the **queue preserves order**.

---

### Why this works

- React maintains an **update queue** for each state hook.
    
- Each update is either:
    
    - a **value update** (replace state), or
        
    - a **function update** (compute new state from old).
        
- React processes them in FIFO order (first in, first out).
    

---

### Example to prove it

```js
const [count, setCount] = useState(0);

function handleClick() {
  setCount(1);                     // update 1
  setCount(c => c + 1);            // update 2
  setCount(c => c * 10);           // update 3
}

// After one click → count = 20
```

👉 Why 20?

- Start: `count = 0`
    
- Update 1: set to `1`
    
- Update 2: run `(1 + 1) = 2`
    
- Update 3: run `(2 * 10) = 20`
    

All in **order**.

---

✅ So yes:  
React batches for performance, but within a single render cycle, **state updates are guaranteed to be applied in the same order you wrote them**.