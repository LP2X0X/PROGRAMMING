---
tags: 
 - react
 - router
 - library
 - overview
---

**React Router** is a powerful library for handling **routing** in React applications — i.e., deciding **which component** to render based on the **URL**.

---

## 🚀 1️⃣ What React Router Does

React Router lets you:

- Create **multiple pages** in a single-page app (SPA)
    
- Navigate between pages **without full page reload**
    
- Manage **URLs**, **parameters**, and **query strings**
    
- Handle **protected routes**, **nested routes**, and **redirects**
    

It keeps your UI in sync with the browser’s URL.

---

## 🧩 2️⃣ Core Concepts

### 🧭 Routes

Each route defines a **path** and the **component** to render:

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./Home";
import About from "./About";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

- `BrowserRouter`: wraps your entire app and manages navigation using the browser’s history API.
    
- `Routes`: groups your routes.
    
- `Route`: maps a URL path → component.
    

---

### 🧭 3️⃣ Navigation ([[Link vs NavLink]])

There are two main ways to navigate between routes:

#### 🖱️ 1. Declarative navigation (with `<Link>`):

```jsx
import { Link } from "react-router-dom";

<Link to="/about">Go to About</Link>
```

✅ Does **not reload** the page.

#### ⚙️ 2. Programmatic navigation (with `useNavigate()`):

```jsx
import { useNavigate } from "react-router-dom";

function Home() {
  const navigate = useNavigate();
  return <button onClick={() => navigate("/about")}>Go</button>;
}
```

✅ Useful when navigation depends on logic (e.g., form submission or API result).

---

## 🧱 4️⃣ Dynamic Routing ([[Dynamic Route]]) 

You can define **route parameters** using `:paramName`:

```jsx
<Route path="/users/:userId" element={<UserProfile />} />
```

Access it inside your component:

```jsx
import { useParams } from "react-router-dom";

function UserProfile() {
  const { userId } = useParams();
  return <h2>User ID: {userId}</h2>;
}
```

If you visit `/users/42`, it renders `User ID: 42`.

---

## 🔍 5️⃣ Query Strings ([[Manipulating Query String]])

React Router doesn’t parse query strings automatically,  
but you can use `useSearchParams`:

```jsx
import { useSearchParams } from "react-router-dom";

function SearchPage() {
  const [params, setParams] = useSearchParams();
  const term = params.get("q"); // ?q=keyword

  return <p>Search for: {term}</p>;
}
```

---

## 🧩 6️⃣ Nested Routes ([[Nested Route]])

You can nest routes inside each other to represent hierarchy.

```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route path="stats" element={<Stats />} />
  <Route path="settings" element={<Settings />} />
</Route>
```

Inside `Dashboard.jsx`:

```jsx
import { Outlet } from "react-router-dom";

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet /> {/* Nested route content renders here */}
    </div>
  );
}
```

---

## 🔒 7️⃣ Protected Routes

You can guard certain routes (e.g., only logged-in users):

```jsx
function ProtectedRoute({ children }) {
  const isLoggedIn = localStorage.getItem("token");
  return isLoggedIn ? children : <Navigate to="/login" />;
}
```

Usage:

```jsx
<Route path="/profile" element={<ProtectedRoute><Profile /></ProtectedRoute>} />
```

---

## ⏩ 8️⃣ Redirecting

You can redirect users using `<Navigate>`:

```jsx
<Route path="/" element={<Navigate to="/home" replace />} />
```

---

## 🧰 9️⃣ Useful Hooks Summary

|Hook|Purpose|
|---|---|
|`useNavigate()`|Navigate programmatically|
|`useParams()`|Read dynamic route params|
|`useLocation()`|Access the current URL (pathname, search, etc.)|
|`useSearchParams()`|Read and write query string parameters|
|`useRoutes()`|Define routes as objects instead of JSX|
|`useOutletContext()`|Pass data between parent and nested routes|

---

## 🧱 1️⃣0️⃣ Route Layouts

You can use nested routes to create **layouts** (shared header/sidebar):

```jsx
<Route path="/" element={<Layout />}>
  <Route index element={<Home />} />
  <Route path="about" element={<About />} />
</Route>
```

Where `Layout` includes:

```jsx
function Layout() {
  return (
    <div>
      <Header />
      <Outlet /> {/* Where the nested page appears */}
    </div>
  );
}
```

---

## ⚙️ 1️⃣1️⃣ HashRouter vs BrowserRouter

|Type|URL Format|Use Case|
|---|---|---|
|`BrowserRouter`|`/about`|Default, uses HTML5 history API|
|`HashRouter`|`#/about`|For static file servers without proper backend routing support|

---

## ✅ Summary

|Concept|Description|
|---|---|
|Router|Wraps app and manages navigation|
|Route|Maps a URL path to a component|
|Link|Navigates declaratively|
|useNavigate|Navigate imperatively (programmatically)|
|useParams|Access dynamic URL params|
|useSearchParams|Handle query strings|
|Outlet|Render nested routes|
|Navigate|Redirect user|
