---
tags: 
 - react
 - effect
 - example
 - note
---

### 🧩 Step 1: What happens without cleanup

Your effect:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setTime((t) => {
      if (t === 0) {
        clearInterval(id);
        return 0;
      }
      return t - 1;
    });
  }, 1000);
}, []);
```

Looks fine, right?  
But here’s the catch 👇

---

### 🧱 Step 2: React Strict Mode behavior (in development)

By default, most React apps (especially Vite, CRA, Next.js) have **Strict Mode** enabled.

In Strict Mode (development only):

1. React mounts your component.
    
2. Runs your `useEffect`.
    
3. Then **immediately unmounts** the component.
    
4. Then **mounts it again** — to help detect side effects that aren’t properly cleaned up.
    

This means:

- Your effect runs **twice**.
    
- So you call `setInterval()` **twice**.
    
- You now have **two active intervals** running in parallel.
    

And because you never cleaned them up, both intervals call `setTime()` every second.  
That’s why your timer looks like it’s “going -2 per second” or even faster if re-rendering stacks up.

---

### 📉 Example visualization

|Time (seconds)|Intervals active|State change|
|---|---|---|
|0|1 (first mount)|—|
|0.1|2 (after re-mount)|—|
|1|both intervals run → `t = t - 2`||
|2|both run again → `t = t - 2`||
|…|keeps doubling if re-rendered again||

So your timer doesn’t “know math wrong” — it’s being told twice (or more) per second to decrement. 😅

---

### 🧩 Step 3: Why cleanup fixes it

When you add:

```jsx
return () => clearInterval(id);
```

React can properly clean up the old interval when:

- The component unmounts (e.g. during Strict Mode test)
    
- Or the effect re-runs (if dependencies change)
    

This ensures that only **one** active interval exists at any given time.

So in Strict Mode:

- React mounts → sets interval A
    
- React unmounts → cleanup runs → clears interval A ✅
    
- React re-mounts → sets interval B (fresh one)
    
- Now only interval B runs → perfect 1-second countdown
    

---

### 🧠 Step 4: Production note

If you build your app (`npm run build`), React **removes** that double-mount behavior in production mode — so it _might_ “look fine” even without cleanup.

But relying on that is bad practice, because:

- It can still leak memory or timers.
    
- If the effect ever re-runs (dependencies change), you’ll get duplicated intervals again.
    

---

### ✅ TL;DR

|Problem|Cause|
|---|---|
|Timer decreases too fast|Multiple intervals created by Strict Mode|
|Why multiple intervals?|You didn’t clear the old one (`clearInterval`)|
|Fix|Always return a cleanup function from your `useEffect`|

---

Would you like me to show a simple **console log timeline** example that visually shows how multiple intervals overlap without cleanup?