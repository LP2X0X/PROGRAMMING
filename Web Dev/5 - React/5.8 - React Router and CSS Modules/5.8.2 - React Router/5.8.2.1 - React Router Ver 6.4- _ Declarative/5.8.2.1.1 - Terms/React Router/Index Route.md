---
tags: 
 - react
 - index
 - route
---

An **index route** in React Router (v6+) is a **default child route** that renders when no other child route matches.

It’s like saying:

> “If you’re at the parent path but haven’t gone deeper, show this.”

---

### 🧭 How React Router interprets `/`

When you write:

```jsx
<Route index element={<Homepage />} />
```

It’s **shorthand** for:

```jsx
<Route path="/" element={<Homepage />} />
```

That’s because the **`index` route** inside your `<Routes>` acts as the “default” or “home” route of that router context.

So when your `<Link to="/" />` runs, React Router:

1. Looks at your routes.
    
2. Sees that the **index route** (`<Route index ... />`) matches `/`.
    
3. Renders the `<Homepage />` component.
    

---

### 🔹 Example

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Dashboard from "./Dashboard";
import Profile from "./Profile";
import Settings from "./Settings";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="dashboard" element={<Dashboard />}>
          {/* 👇 Index route */}
          <Route index element={<p>Welcome to the dashboard!</p>} />
          <Route path="profile" element={<Profile />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

---

### 🔹 Dashboard (parent layout)

```jsx
import { Outlet, Link } from "react-router-dom";

export default function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
      <nav>
        <Link to="">Home</Link> |{" "}
        <Link to="profile">Profile</Link> |{" "}
        <Link to="settings">Settings</Link>
      </nav>

      {/* Nested route content goes here */}
      <Outlet />
    </div>
  );
}
```

---

### 🔹 How it behaves

- `/dashboard` → shows `Welcome to the dashboard!`
    
- `/dashboard/profile` → shows `<Profile />`
    
- `/dashboard/settings` → shows `<Settings />`
    

---

✅ **Key point:**  
An index route has **no path**. You just write `<Route index element={...} />`.