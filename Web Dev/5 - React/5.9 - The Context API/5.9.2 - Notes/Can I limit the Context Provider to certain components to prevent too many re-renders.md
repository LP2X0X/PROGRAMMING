---
tags: 
 - web dev
---

## 🧭 TL;DR

✅ **Yes, you can wrap context providers locally** to avoid unnecessary re-renders and large refresh boundaries.
⚠️ But doing so changes how your context behaves — it limits its *scope* and *shared state*.

---

## ⚙️ How Context Providers Work

Every time you wrap a component in a context provider:

```jsx
<MyContext.Provider value={someValue}>
  <SomeComponent />
</MyContext.Provider>
```

…you’re creating a **new isolated context instance** — a new "bubble" where consumers read from.

So:

* Components **inside** this bubble use the new `value`.
* Components **outside** it see nothing (or a different provider’s value).

This means that *local wrapping* is totally valid — it just narrows scope and changes re-render patterns.

---

## 🧩 Why Wrapping Locally Can Be Good

### ✅ 1. Smaller refresh / re-render scope

If your provider lives at the top level (like `<App />`), any update to its value causes **everything below it** to re-render.

If you wrap it **closer** to where it’s needed:

```jsx
function Page() {
  return (
    <UserProvider>
      <UserProfile />
    </UserProvider>
  );
}
```

Then only `UserProfile` (and its descendants) re-render when context changes.

This is great for performance **and** for development — less state loss on Fast Refresh.

---

### ✅ 2. Easier local reasoning

Local providers help isolate concerns.
For example:

```jsx
function SettingsPage() {
  return (
    <ThemeProvider>
      <SettingsForm />
    </ThemeProvider>
  );
}
```

`SettingsPage` now controls its own theme state, without affecting the global app theme.

---

## ⚠️ Trade-offs / Gotchas

| Issue                 | Description                                                                                                                                                            |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🧠 Separate instances | Each local provider has its own state — two `UserProviders` don’t share data.                                                                                          |
| 🔁 Value recreation   | If you create the provider inside a component and compute `value` inline, React might re-create the context value on each render (causing extra renders in consumers). |
| 🧩 Nested providers   | Deeply nesting multiple contexts can make your tree harder to debug.                                                                                                   |
| ♻️ React Fast Refresh | Wrapping providers locally can indeed **reduce how much of your app re-renders on refresh**, because the provider tree resets only where it’s declared — not globally. |

---

## 🪄 Best Practice Pattern

If you want to **avoid global wrapping but keep shared logic**, try this hybrid pattern:

```jsx
// context/MyContext.jsx
import { createContext, useContext, useState, useMemo } from 'react';

const MyContext = createContext();

export function MyProvider({ children }) {
  const [count, setCount] = useState(0);
  const value = useMemo(() => ({ count, setCount }), [count]);
  return <MyContext.Provider value={value}>{children}</MyContext.Provider>;
}

export function useMyContext() {
  const ctx = useContext(MyContext);
  if (!ctx) throw new Error('useMyContext must be used within MyProvider');
  return ctx;
}
```

Then use it **locally**:

```jsx
function Dashboard() {
  return (
    <MyProvider>
      <UserStats />
    </MyProvider>
  );
}
```

Now only the dashboard tree re-renders when the context changes — and Fast Refresh won’t reset unrelated parts of your app.

---

## ✅ When to Wrap Locally vs Globally

| Use Case                            | Best Practice                              |
| ----------------------------------- | ------------------------------------------ |
| **Global themes, routing, auth**    | Wrap high up (once per app)                |
| **Page- or feature-specific state** | Wrap locally, near the component           |
| **Performance-sensitive parts**     | Wrap locally to reduce unnecessary updates |