---
tags: 
 - react
 - router
 - overview
---

## 🚀 1. Install React Router

```bash
npm install react-router-dom
```

---

## 🏗️ 2. Basic Setup

In your project, typically you’ll wrap your app with a `BrowserRouter` at the root.

**`main.jsx` (or `index.js`):**

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

---

## 🧩 3. Define Routes

In `App.jsx`, define routes using `<Routes>` and `<Route>`.

**`App.jsx`:**

```jsx
import { Routes, Route, Link } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import NotFound from "./pages/NotFound";

export default function App() {
  return (
    <div>
      <nav>
        <Link to="/">🏠 Home</Link> | <Link to="/about">ℹ️ About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        {/* Catch unmatched routes */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </div>
  );
}
```

---

## 📁 4. Example Pages

**`pages/Home.jsx`:**

```jsx
export default function Home() {
  return <h1>Welcome to the Home Page!</h1>;
}
```

**`pages/About.jsx`:**

```jsx
export default function About() {
  return <h1>About Us</h1>;
}
```

**`pages/NotFound.jsx`:**

```jsx
export default function NotFound() {
  return <h1>404 - Page Not Found 😢</h1>;
}
```

---

## 🔄 5. Dynamic Routes (Params)

You can use route parameters like `/movies/:id`.

**`App.jsx`:**

```jsx
<Route path="/movies/:id" element={<MovieDetail />} />
```

**`pages/MovieDetail.jsx`:**

```jsx
import { useParams } from "react-router-dom";

export default function MovieDetail() {
  const { id } = useParams();
  return <h1>Movie ID: {id}</h1>;
}
```

Visiting `/movies/123` will display `Movie ID: 123`.

---

## 🪄 6. Navigate Programmatically

Use `useNavigate()` to go to another page by code (e.g., after form submit).

```jsx
import { useNavigate } from "react-router-dom";

function Login() {
  const navigate = useNavigate();

  function handleLogin() {
    // ...perform login
    navigate("/dashboard");
  }

  return <button onClick={handleLogin}>Login</button>;
}
```

---

## 🧭 7. Nested Routes

Nested routes allow layouts that share common UI parts.

**`App.jsx`:**

```jsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Home />} />
    <Route path="about" element={<About />} />
    <Route path="contact" element={<Contact />} />
  </Route>
</Routes>
```

**`Layout.jsx`:**

```jsx
import { Outlet, Link } from "react-router-dom";

export default function Layout() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link> | <Link to="/about">About</Link> | <Link to="/contact">Contact</Link>
      </nav>
      <Outlet /> {/* renders the child route here */}
    </div>
  );
}
```

---

## 💡 8. Use Search Params (Query Strings)

```jsx
import { useSearchParams } from "react-router-dom";

function Movies() {
  const [searchParams, setSearchParams] = useSearchParams();
  const filter = searchParams.get("filter") || "all";

  return (
    <>
      <h2>Showing: {filter}</h2>
      <button onClick={() => setSearchParams({ filter: "popular" })}>Popular</button>
    </>
  );
}
```

Visiting `/movies?filter=popular` will show `Showing: popular`.

---

## ⚙️ 9. Code Splitting with Lazy Loading

React Router works great with React’s `lazy` and `Suspense`:

```jsx
import { lazy, Suspense } from "react";
import { Routes, Route } from "react-router-dom";

const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));

export default function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  );
}
```

---

## ✅ 10. Summary Table

| Feature              | Hook / Component            | Example                                 |
| -------------------- | --------------------------- | --------------------------------------- |
| Basic route setup    | `<BrowserRouter>`           | Wrap app                                |
| Define routes        | `<Routes>`, `<Route>`       | `<Route path="/" element={<Home />} />` |
| Navigation (links)   | `<Link>` / `<NavLink>`      | `<Link to="/about">About</Link>`        |
| Dynamic route params | `useParams()`               | `/users/:id`                            |
| Navigate in code     | `useNavigate()`             | `navigate("/home")`                     |
| Nested routes        | `<Outlet>`                  | Layout pages                            |
| Query params         | `useSearchParams()`         | `/movies?filter=top`                    |
| Lazy loading         | `React.lazy()` + `Suspense` | Async route components                  |