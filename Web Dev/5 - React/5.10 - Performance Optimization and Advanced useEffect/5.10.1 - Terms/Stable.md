---
tags: 
 - react
 - term
 - memoization
---

In React, a value is **stable** if its **reference (identity)** does **not change between renders**, unless its **content truly changes**.

In JavaScript, **objects, arrays, and functions** are compared by _reference_, not by _value_.

```js
{} === {} // false — two different object references
```

So if you create a new object on each render:

```jsx
<ThemeContext.Provider value={{ color: "red" }}>
```

Even though the data looks the same (`{ color: "red" }`), React sees it as **a new object reference** every render — meaning **unstable**.