---
tags: 
 - react
 - router
 - navigate
---

## 🧩 What `<Navigate />` does

It’s a special React Router component that **redirects** the user to another route.

Think of it as the declarative version of `useNavigate()` —  
instead of calling a function, you just _render_ `<Navigate />`, and React Router changes the route.

---

## 🧭 1. Basic usage

```jsx
import { Navigate } from "react-router-dom";

function Home() {
  return <Navigate to="/login" />;
}
```

✅ When this component renders, the user is **immediately redirected** to `/login`.

---

## ⚙️ 2. Conditional redirect

Common use case: protect routes (redirect if not logged in).

```jsx
import { Navigate } from "react-router-dom";

function ProtectedRoute({ user, children }) {
  if (!user) return <Navigate to="/login" replace />;
  return children;
}
```

Then use it:

```jsx
<Route
  path="dashboard"
  element={
    <ProtectedRoute user={user}>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

✅ If `user` is falsy, React Router sends them to `/login`.

---

## 🔁 3. The `replace` prop

```jsx
<Navigate to="/home" replace />
```

✅ Replaces the current entry in the browser history stack.  

With `replace`, it _pushes_ a new entry, so Back would return to the old page.

---

## 🧠 4. Using `Navigate` with state

You can pass state just like with `<Link>`:

```jsx
<Navigate to="/profile" state={{ from: "login" }} />
```

Then retrieve it in the target component:

```jsx
import { useLocation } from "react-router-dom";

function Profile() {
  const location = useLocation();
  console.log(location.state.from); // "login"
}
```

---

## 🧩 5. Difference from `useNavigate()`

|Hook / Component|How it works|Example|
|---|---|---|
|`useNavigate()`|Imperative (you _call_ it)|`navigate("/login")`|
|`<Navigate />`|Declarative (you _render_ it)|`<Navigate to="/login" />`|

✅ Use `<Navigate />` inside JSX (e.g. as a conditional render).  
✅ Use `useNavigate()` inside event handlers or effects.

---

Perfect — great idea 👍  
The `<Navigate />` component indeed has an especially **good use case** when used with **index routes**.  
Let’s add that as its own section 👇

---

## 🧩 6. Best use case — with `index` routes

When you define an **index route** inside a nested `<Route>`, you can use `<Navigate />` to **redirect the default (index) path** to a more specific one.

### 🧭 Example

```jsx
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import AppLayout from "./AppLayout";
import Dashboard from "./Dashboard";
import Reports from "./Reports";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="app" element={<AppLayout />}>
          {/* 👇 Default redirect when visiting /app */}
          <Route index element={<Navigate to="dashboard" replace />} />

          <Route path="dashboard" element={<Dashboard />} />
          <Route path="reports" element={<Reports />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### 🧠 What happens

- If the user visits `/app`,  
    React Router automatically redirects them to `/app/dashboard`.
    
- This makes sure your nested route always loads a _default subpage_.
    

### ✅ Why this is the _best use case_

- Keeps your routing **clean** — no extra logic needed.
    
- Prevents blank pages on index routes.
    
- Works declaratively, without `useNavigate()` or effects.
    

---

## 💡 Example: Conditional redirect after login

```jsx
function Login({ user }) {
  if (user) return <Navigate to="/dashboard" replace />;
  return <LoginForm />;
}
```

✅ As soon as `user` exists, it redirects to `/dashboard`.