---
tags:
  - react
  - term
  - fundamental
---

- In React, **Fiber node** (or Fiber for short) is the smallest building block of React’s Fiber architecture — it’s a plain JavaScript object that stores everything React needs to know about a component or element in your UI.
- Think of it as a **data record** for one “unit of work” in React’s rendering process.

```ad-note
Fibers are not re-created on every render.
```

- **Purpose:** Keeps track of:
    - The component’s type and props
    - Its state and hooks
    - Where it is in the UI tree
    - Side effects that need to run
    - Links to its parent, child, and siblings
    - Queue of work
