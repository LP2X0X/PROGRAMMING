---
tags: 
 - react
 - component
 - re-render
---

## ☝️ **Props change does not cause re-render**

### 💡 What really happens

1. **Props only changes when the parent component re-renders.**  
    
2. When the parent re-renders, **the children who receives the prop will re-render anyway!**
    
3. Therefore, the real reason why a component re-renders when props change is **because the parent re-renders**.
    

### 🧩 Summary

- Props don’t change magically — they’re recalculated **when the parent re-renders**.
    
- **Props “change” only if the parent re-renders**, and that’s when the child will re-render (unless memoized).
