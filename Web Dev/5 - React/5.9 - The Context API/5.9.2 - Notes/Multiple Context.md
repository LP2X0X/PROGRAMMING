---
tags: 
 - react
 - context
 - api
 - note
---

You can create and use **multiple Contexts** in React — it’s common for apps to separate concerns (e.g., theme, auth, settings).

Let’s go step by step 👇

---

## 🧩 1️⃣ Create multiple contexts

Each context is created independently:

```jsx
import { createContext } from "react";

export const ThemeContext = createContext();
export const AuthContext = createContext();
```

---

## ⚙️ 2️⃣ Provide them separately

You can **nest multiple Providers** — each one gives its value to children.

```jsx
import React, { useState } from "react";
import { ThemeContext, AuthContext } from "./contexts";

function App() {
  const [theme, setTheme] = useState("light");
  const [user, setUser] = useState(null);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <AuthContext.Provider value={{ user, setUser }}>
        <Main />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```

✅ Now everything inside `<Main />` has access to **both contexts**.

---

## 📥 3️⃣ Consume multiple contexts

In any component, just call `useContext()` for each one:

```jsx
import { useContext } from "react";
import { ThemeContext, AuthContext } from "./contexts";

function Profile() {
  const { theme } = useContext(ThemeContext);
  const { user } = useContext(AuthContext);

  return (
    <div className={theme === "dark" ? "dark" : "light"}>
      {user ? `Welcome, ${user.name}` : "Please log in"}
    </div>
  );
}
```

---

## 🧠 4️⃣ Optional: combine with custom hook for cleaner code

If you often need both contexts together, you can make a **custom hook**:

```jsx
function useAppContexts() {
  const theme = useContext(ThemeContext);
  const auth = useContext(AuthContext);
  return { ...theme, ...auth };
}
```

Now just:

```jsx
const { theme, user } = useAppContexts();
```

---

## ✅ Summary

|Step|What to do|
|---|---|
|1️⃣|Create multiple contexts with `createContext()`|
|2️⃣|Wrap components in multiple Providers|
|3️⃣|Use `useContext()` separately for each|
|4️⃣|Optionally combine them with a custom hook for convenience|