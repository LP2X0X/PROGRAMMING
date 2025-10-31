---
tags: 
 - react
 - nested
 - route
---

A **nested route** means:

> One route (child) lives **inside another route (parent)** —  
> both share part of the same layout or page structure.

So instead of having each page be completely separate, you can **reuse layout components** like navbars, sidebars, etc.

```ad-note
We need this when we want a part of the user interface to be controlled by a part of the URL. This is what defined a nested route, not because it is made up of multiple parts in the URL.
```

---

## 🏗 Example: Without Nested Routes

You might write this:

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/dashboard/settings" element={<Settings />} />
</Routes>
```

This works — but `Dashboard` and `Settings` don’t share layout or state.  
They render separately.

---

## 🌿 With Nested Routes

You can group related routes like this:

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/dashboard" element={<DashboardLayout />}>
    <Route index element={<DashboardHome />} />
    <Route path="settings" element={<Settings />} />
  </Route>
</Routes>
```

Now:

- `/dashboard` → renders `DashboardLayout` + `DashboardHome`
    
- `/dashboard/settings` → renders `DashboardLayout` + `Settings`
    

Both share the same **layout component**!

---

## 🧭 How It Works Internally

1. React Router matches `/dashboard/settings`
    
2. It renders `<DashboardLayout>` first
    
3. Then it renders the matching **child** route (`<Settings />`) **inside** the `<Outlet />`
    

---

## ✅ Benefits

|Benefit|Description|
|---|---|
|**Shared layout**|Keep navbars, sidebars, or headers across multiple pages|
|**Cleaner structure**|Group related routes logically|
|**Scoped URLs**|`/dashboard/*` clearly defines a section of your app|
|**Reusability**|Layouts can wrap multiple pages without duplication|

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