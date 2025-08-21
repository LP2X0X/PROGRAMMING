---
tags: 
 - js
 - function
 - technique
---

#### Requirement

- The goal is to create a **throttle decorator**. The **wrapper function** should ensure:
	- **First call**: `f` is invoked **immediately** on the very first call.
	- **During cooldown** (within `ms` milliseconds after the call):
	    - New calls are **ignored** — you don’t call `f` again immediately.
	    - However, the **last call’s arguments and `this` context** should be _saved_.
	- **After cooldown ends**:
	    - If there was at least one call during the cooldown, invoke `f` **once more** using the **last saved arguments and context**.
	    - Then start a new cooldown cycle.
	- This ensures **at most one call per `ms` milliseconds**, while still handling the most recent input.

---

#### Solution

```js
function throttle(f, ms) {
  let isThrottle = false;
  let savedArgs;
  let savedThis;

  function wrapper(...args) {
    if (isThrottle) {
      savedArgs = args; // Need to save both args because if we call wrapper.apply(this, args) it will always call with data from the first call only, not the last
      savedThis = this; //                   this
      return;
    }

    // When the function wrapper.apply called, it will get this and args from the outer lexical environment
    // at that moment

    isThrottle = true;

    f.apply(this, args); // First call since start of cycle

    setTimeout(() => {
      isThrottle = false;
      if (savedArgs) {
        // Condition to stop the throttle
        wrapper.apply(savedThis, savedArgs); // Call after ms time recursively to start new cycle
        // Prepare for new cycle.
        savedArgs = null;
        savedThis = null;
      }
    }, ms);
  }

  return wrapper;
}
```