---
tags: 
 - react
 - note
 - strict
 - mode
---

# 1. What React Strict Mode does in React 18

React 18 introduces a development-only behavior:

**When a component mounts for the first time, Strict Mode:**

1. Mounts the component
    
2. Runs the effect
    
3. Immediately unmounts the component
    
4. Immediately _remounts_ it again
    
5. Restores state on the second mount
    
6. Runs the effect again
    

This is called **"double-invocation of mount lifecycles"**.

Production mode does **not** do this.

---

# 2. Why this helps catch bugs

## The root reason:

**Many bugs happen because developers write effects that assume the component mounts only once.  
But in real apps (Suspense, React.lazy, fast refresh, concurrent rendering), components CAN be mounted, unmounted, and re-mounted multiple times.**

React wants to make these issues visible in development before they cause real bugs.

---

# 3. What sorts of bugs does this detect?

Here are the **three most common issues**.

---

## Bug 1 — Effects that rely on running only once

Example: making an API request inside `useEffect` without cleanup.

```js
useEffect(() => {
  fetch("/api/start-session");
}, []);
```

If this component mounts twice due to concurrent rendering, the request runs twice.  
Strict Mode exposes that by doing a fake mount → unmount → remount.

**Why this is a problem:**

- Production may also remount during transitions
    
- Double network requests may cause data duplication or unintended side effects
    

---

## Bug 2 — Missing cleanup for subscriptions, timers, or listeners

Example:

```js
useEffect(() => {
  const id = setInterval(() => console.log("tick"), 1000);
}, []);
```

This effect has **no cleanup**.  
If the component is unmounted and re-mounted, old intervals keep running.

Strict Mode forces:

- one mount → interval created
    
- unmount → cleanup SHOULD remove it
    
- remount → new interval created
    

But since cleanup doesn’t clear it, Strict Mode causes TWO intervals to run.

You immediately see:

- duplicated logs
    
- memory leaks
    
- repeating actions
    

This exposes the forgotten cleanup.

Correct version:

```js
useEffect(() => {
  const id = setInterval(() => console.log("tick"), 1000);
  return () => clearInterval(id);
}, []);
```

---

## Bug 3 — Effects that modify external systems incorrectly

Example: adding a DOM event listener but forgetting to remove it.

```js
useEffect(() => {
  window.addEventListener("scroll", handler);
}, []);
```

Strict Mode mount/unmount/remount reveals:

- the original listener is never removed
    
- the second mount adds a second identical listener
    
- your handler fires twice per scroll
    

This is exactly the kind of bug that would quietly cause performance issues in production.

---

# 4. Why double-remounting is such an effective detector

Because **the only correct effects are those that:**

- don’t produce side effects that depend on running only once
    
- always clean up external subscriptions
    
- remain safe even if mounted/unmounted multiple times
    

React cannot rely on developers writing perfect code.  
Instead, it SIMULATES real-world situations where:

- components mount twice
    
- async rendering aborts a render
    
- transitions interrupt UI
    
- Suspense retries rendering
    

Strict Mode makes these situations visible immediately in development.

---

# 5. Summary in one sentence

React 18 unmounts and remounts components in Strict Mode to **force your effects to behave correctly in real concurrent rendering**, uncovering bugs related to missing cleanup, double side effects, or unsafe mount assumptions.
