---
tags: 
 - react
 - nested
 - route
---

In **React Router (v6+)**, nested routes let you build hierarchical layouts where a parent route renders some layout, and child routes get rendered inside it.

```ad-note
We need this when we want a part of the user interface to be controlled by a part of the URL. This is what defined a nested route, not because it is made up of multiple parts in the URL.
```

---

### 1. Basic Setup

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import Dashboard from "./pages/Dashboard";
import Profile from "./pages/Profile";
import Settings from "./pages/Settings";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Top-level routes */}
        <Route path="/" element={<Home />} />
        
        {/* Parent route */}
        <Route path="/dashboard" element={<Dashboard />}>
          {/* Nested routes */}
          <Route path="profile" element={<Profile />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

---

### 2. Dashboard Component (the parent layout)

For the nested routes to appear, you must render an `<Outlet />` inside the parent:

```jsx
import { Outlet, Link } from "react-router-dom";

export default function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
      <nav>
        <Link to="profile">Profile</Link> |{" "}
        <Link to="settings">Settings</Link>
      </nav>
      
      {/* Where nested routes render */}
      <Outlet />
    </div>
  );
}
```

---

### 3. How It Works

- Going to `/dashboard` → shows `<Dashboard>` only.
    
- Going to `/dashboard/profile` → shows `<Dashboard>` with `<Profile>` rendered in `<Outlet>`.
    
- Going to `/dashboard/settings` → shows `<Dashboard>` with `<Settings>` rendered in `<Outlet>`.
    

---

### 4. Index Route (default child)

If you want something to show when no nested path is chosen:

```jsx
<Route index element={<p>Welcome to the dashboard!</p>} />
```

So inside `Dashboard`’s children:

```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route index element={<p>Welcome to the dashboard!</p>} />
  <Route path="profile" element={<Profile />} />
  <Route path="settings" element={<Settings />} />
</Route>
```