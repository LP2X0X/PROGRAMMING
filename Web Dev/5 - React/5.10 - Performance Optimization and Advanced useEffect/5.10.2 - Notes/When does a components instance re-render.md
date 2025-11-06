---
tags: 
 - react
 - component
 - re-render
---

## ☝️ **Props change does not cause re-render**

### 💡 What really happens

1. **Props themselves don’t trigger re-render.**  
    React components re-render when **their parent re-renders**.
    
2. When the parent re-renders, React **re-evaluates the JSX** and passes **new props** to its children.
    
3. If a child receives **new prop values** (based on reference or primitive comparison), React will **re-render that child** to reflect the changes.
    

### 🧩 Summary

- Props don’t change magically — they’re recalculated **when the parent re-renders**.
    
- **Props “change” only if the parent re-renders**, and that’s when the child will re-render (unless memoized).
