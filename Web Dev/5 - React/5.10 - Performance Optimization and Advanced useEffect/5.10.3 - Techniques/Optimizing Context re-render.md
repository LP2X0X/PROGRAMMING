---
tags: 
 - react
 - optimizing
 - context
---

## **Optimizing React Context with `useMemo`**

### **Problem**

When a parent component re-renders, any `Context.Provider` it contains will also re-render its consumers **if the `value` prop changes**. Even if the actual state hasn’t changed, creating a new object inline causes unnecessary re-renders.

```tsx
// ❌ Problem: new object created on every render
<ThemeContext.Provider value={{ theme, setTheme }}>
  {children}
</ThemeContext.Provider>
```

Here, `{ theme, setTheme }` is a **new object on every render**, so all consuming components will re-render even if `theme` didn’t change.

```ad-note
If you inspect the process with profiler, you can see the reason for re-render of consumers components are "Context changed".
```

---

### **Solution: Use `useMemo`**

Wrap the `value` in `useMemo` so it only changes when one of its dependencies changes:

```tsx
import { createContext, useState, useMemo } from "react";

const ThemeContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  // ✅ Memoize value to avoid unnecessary re-renders
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

- `useMemo` ensures the object reference stays **stable** unless `theme` actually changes.
    
- Components consuming `ThemeContext` **won’t re-render unnecessarily** on parent re-renders.
    

---

### **Tips**

1. Always memoize objects or functions passed as `value`.
    
2. Consider splitting context if it has multiple unrelated state values.
    
3. Combine with `React.memo` for consumer components for extra optimization.
    

---

✅ **Key takeaway:**  
**Memoizing the `value` prop prevents unwanted re-renders of context consumers when the parent component re-renders.**