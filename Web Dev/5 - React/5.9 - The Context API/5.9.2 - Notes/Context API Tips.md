---
tags: 
 - react
 - context
 - api
 - tips
---

React’s **Context API** allows you to share state or functions across your component tree without passing props manually.  
By combining it with **custom hooks**, you make your context usage cleaner, safer, and easier to maintain.
Here’s an **advanced React pattern** for creating a **custom provider** + **custom hook** with good **type safety, modularity, and developer experience**.

---

## 🧩 Goal

We want a **self-contained pattern**:

- `MyProvider` — defines context and manages state.
    
- `useMyContext()` — safe, clean, and throws helpful errors if misused.
    
- Reusable and composable in large apps.
    

---

## ✅ **Final Pattern (Advanced Version)**

```jsx
// MyProvider.jsx
import { createContext, useContext, useState, useMemo } from "react";

// 1️⃣ Create Context (initialized as undefined for safety)
const MyContext = createContext(undefined);

// 2️⃣ Create Provider component
export function MyProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");

  // Use useMemo to avoid unnecessary re-renders
  const value = useMemo(
    () => ({
      user,
      setUser,
      theme,
      toggleTheme: () => setTheme((t) => (t === "light" ? "dark" : "light")),
    }),
    [user, theme]
  );

  return <MyContext.Provider value={value}>{children}</MyContext.Provider>;
}

// 3️⃣ Create custom hook (safe accessor)
export function useMyContext() {
  const context = useContext(MyContext);
  if (!context) {
    throw new Error("useMyContext must be used within a MyProvider");
  }
  return context;
}
```

---

## 📦 **Usage Example**

```jsx
import { MyProvider, useMyContext } from "./MyProvider";

function Profile() {
  const { user, setUser, theme, toggleTheme } = useMyContext();

  return (
    <div>
      <h2>Theme: {theme}</h2>
      <button onClick={toggleTheme}>Toggle Theme</button>
      <button onClick={() => setUser({ name: "Alice" })}>Login</button>
      {user && <p>Welcome, {user.name}!</p>}
    </div>
  );
}

export default function App() {
  return (
    <MyProvider>
      <Profile />
    </MyProvider>
  );
}
```

---

## 🧠 **Advanced Tips**

### 1. 🔒 Context Factory Pattern

To make it reusable across multiple contexts:

```jsx
// contextFactory.js
import { createContext, useContext } from "react";

export function createCtx(defaultValue) {
  const ctx = createContext(defaultValue);
  function useCtx() {
    const c = useContext(ctx);
    if (!c) throw new Error("useCtx must be inside a Provider with a value");
    return c;
  }
  return [useCtx, ctx.Provider];
}
```

Then:

```jsx
const [useAuth, AuthProvider] = createCtx();

export function MyAuthProvider({ children }) {
  const [user, setUser] = useState(null);
  return (
    <AuthProvider value={{ user, setUser }}>
      {children}
    </AuthProvider>
  );
}
```

---

### 2. 🧩 Split Providers (for scalability)

For large apps, separate contexts:

```jsx
<AuthProvider>
  <ThemeProvider>
    <App />
  </ThemeProvider>
</AuthProvider>
```

Each with its own custom hook — keeps re-renders isolated.

---

### 3. ⚙️ Lazy Initialization

If the context value is expensive to compute, use a lazy `useMemo` or custom reducer.

---

### 4. 🧱 Combine with Reducer

For complex logic:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
<MyContext.Provider value={{ state, dispatch }}>
```

Then consume with:

```jsx
const { state, dispatch } = useMyContext();
```

---

### ✅ **Summary**

|Feature|Why it matters|
|---|---|
|`createContext(undefined)`|Ensures runtime error if misused|
|Custom Hook (`useMyContext`)|Clean and safe API|
|`useMemo`|Prevents re-renders|
|Context Factory|Reusable pattern for many contexts|
|Reducer Integration|Scales for complex state|

---

### 🧠 **Why This Pattern Is Cleaner**

|Benefit|Description|
|---|---|
|**Encapsulation**|All logic for reading and validating context is inside the custom hook.|
|**Cleaner imports**|Components only import one hook instead of both the context and `useContext`.|
|**Error safety**|Prevents components from using context outside its Provider.|
|**Separation of concerns**|The Provider focuses on state management; the hook focuses on access.|
|**Easy scaling**|You can have multiple contexts (Auth, Theme, Posts, etc.) with consistent structure.|

---

### 🧱 **Why Wrap with Providers**

Wrapping parts of your app with context Providers keeps data **localized** and avoids **prop drilling**:

```jsx
<AuthProvider>
  <ThemeProvider>
    <SettingsProvider>
      <App />
    </SettingsProvider>
  </ThemeProvider>
</AuthProvider>
```

Each Provider manages its own domain (auth, theme, settings), making the app easier to reason about and refactor.

---

### ✅ **Best Practices**

- Use **one custom hook per context** (`useAuth`, `useTheme`, `usePosts`, etc.).
    
- Always **guard** against missing Providers inside the hook.
    
- Keep Provider logic **focused** — avoid mixing unrelated states.
    
- Group Providers at the root or near where they’re needed.
    
- Prefer **derived state** (computed from existing data) over duplicating values.
    

---

### 🧭 **Summary**

|Concept|Purpose|
|---|---|
|**Context Provider**|Supplies shared state or actions to descendant components|
|**`useContext()`**|Retrieves the nearest context value|
|**Custom Hook**|Wraps `useContext` for clean, consistent, and safe usage|
|**Result**|Cleaner code, better modularity, easier testing, and no prop drilling|