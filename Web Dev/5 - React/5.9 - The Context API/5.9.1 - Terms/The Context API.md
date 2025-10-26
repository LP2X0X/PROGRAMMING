---
tags: 
 - react
 - context 
 - API
---

## 🧭 What is the Context API?

The **Context API** in React lets you **share data across your component tree**  
**without having to pass props manually** at every level.

In short:  
👉 It’s a **way to manage global data** in React.

![[Pasted image 20251007151158.png|center|700]]

---

## 💡 The Problem It Solves

Normally, to pass data from a parent to a deeply nested child, you do **“prop drilling”**:

```jsx
<App>
  <Header user={user} />
    <Navbar user={user} />
      <Profile user={user} />
```

If you have many layers, passing `user` everywhere becomes messy. 😩

That’s where **Context API** helps — it provides a **shared “context”** that all components can read from, no matter how deep they are.

```ad-note
It only provides the context for its children components.
```

---

## ⚙️ How It Works (Step-by-step)

### 1️⃣ Create a Context

```jsx
import { createContext } from "react";

export const ThemeContext = createContext();
```

````ad-note
By convention, **Context variables use UpperCamelCase (PascalCase)** because they represent **components or objects** like React components.

Example:

```jsx
const ThemeContext = createContext();
```

This matches React’s naming style — you’ll later use it as:

```jsx
<ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

So:

- ✅ `ThemeContext` → correct
    
- 🚫 `themeContext` → not conventional
````

---

### 2️⃣ Provide the Context (wrap your app)

```jsx
import React, { useState } from "react";
import { ThemeContext } from "./ThemeContext";

function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

🟢 Everything inside `<ThemeContext.Provider>`  
can now access `{ theme, setTheme }`.

```ad-tip
We can wrap the App component in index.js.
```

---

### 3️⃣ Consume the Context

```jsx
import React, { useContext } from "react";
import { ThemeContext } from "./ThemeContext";

function Toolbar() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
    </div>
  );
}
```

Now `Toolbar` doesn’t need props from its parent — it directly reads from `ThemeContext`.

---

## 🧠 Key Hooks and Components

|Tool|Description|
|---|---|
|`createContext()`|Creates a new context object.|
|`<Context.Provider>`|Makes the context value available to children.|
|`useContext(Context)`|Lets any component read that value.|

---

## ⚡ When Context Value Changes — Consumers Re-render

Whenever the **value** you pass to a context’s `<Provider>` changes, **all components using that context (via `useContext`) will re-render**.

#### 🧩 Example

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Content />
    </ThemeContext.Provider>
  );
}
```

In both `Header` and `Content`:

```jsx
const { theme } = useContext(ThemeContext);
```

👉 When `setTheme("dark")` is called:

- The `value` inside `<ThemeContext.Provider>` changes (`theme` → `"dark"`)
    
- All components using `ThemeContext` **automatically re-render**
    
- They re-run their render function with the updated value
    

#### 🔁 Why This Happens

React compares the **reference** of the value passed to `<Provider>`:

```jsx
<ThemeContext.Provider value={{ theme, setTheme }}>
```

Every time `theme` changes, `{ theme, setTheme }` creates a **new object**, so React sees the `value` prop as **different** and triggers updates.

#### 🧠 Optimization Tip

If your context contains many values or updates frequently, you can **prevent unnecessary re-renders** using `useMemo`:

```jsx
const contextValue = useMemo(() => ({ theme, setTheme }), [theme]);
```

Then:

```jsx
<ThemeContext.Provider value={contextValue}>
```

✅ Now, the object only changes when `theme` changes — not on every render.

---

## 🌎 Common Use Cases

|Use Case|Example|
|---|---|
|Theme management|Dark / light mode|
|Authentication|Current user, login/logout|
|Language|i18n locale|
|Global app state|e.g., app settings, cart data|

---

## 🚫 When Not to Use It

Avoid Context for:

- Frequently changing values (it triggers re-renders in all consumers)
    
- Large complex state (better use libraries like **Redux**, **Zustand**, or **Jotai**)
    

---

## ✅ Summary

|Concept|Description|
|---|---|
|**Goal**|Share data globally without prop drilling|
|**Core Tools**|`createContext`, `Provider`, `useContext`|
|**Best For**|Themes, auth, locale, or small global states|
|**Not Ideal For**|High-frequency updates or very large state trees|