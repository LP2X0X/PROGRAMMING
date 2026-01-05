---
tags: 
 - react
 - router
 - modern
 - how
---

# Modern React Router (v6.4+) — How To

## 1) Install

```bash
# This install react-router-dom which include react-router
npm install react-router-dom
```

This package includes the core router and all browser integrations.

---

## 2) Define Routes as Data (Router Object)

Create a router using `createBrowserRouter`. Routes are **plain objects**, not JSX.

```jsx
// router.js
import { createBrowserRouter } from "react-router";
import RootLayout from "./layouts/RootLayout";
import Home from "./pages/Home";
import About from "./pages/About";
import NotFound from "./pages/NotFound";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <NotFound />,
    children: [
      { index: true, element: <Home /> },
      { path: "about", element: <About /> },
    ],
  },
]);
```

**Why this is modern**

- Routes, data, errors, and code-splitting live together
    
- URL becomes the source of truth
    

---

## 3) Activate the Router

Mount the router once with `RouterProvider`.

```jsx
// main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { RouterProvider } from "react-router-dom";
import { router } from "./router";

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```

No `<BrowserRouter>` is used.

---

## 4) Create a Layout with `<Outlet>`

Layouts render shared UI and a placeholder for child routes.

```jsx
// layouts/RootLayout.jsx
import { Outlet, Link } from "react-router-dom";

export default function RootLayout() {
  return (
    <>
      <nav>
        <Link to="/">Home</Link> | <Link to="/about">About</Link>
      </nav>
      <Outlet />
    </>
  );
}
```

---

## 5) Add Dynamic Routes (Params)

Define params directly in the route path.

```js
{
  path: "movies/:id",
  element: <MovieDetail />,
}
```

```jsx
import { useParams } from "react-router-dom";

export default function MovieDetail() {
  const { id } = useParams();
  return <h1>Movie ID: {id}</h1>;
}
```

---

## 6) Load Data with `loader` (Recommended)

Fetch data **before rendering** the route.

```js
{
  path: "movies/:id",
  element: <MovieDetail />,
  loader: async ({ params }) => {
    const res = await fetch(`/api/movies/${params.id}`);
    if (!res.ok) throw new Response("Not Found", { status: 404 });
    return res.json();
  },
}
```

```jsx
import { useLoaderData } from "react-router-dom";

export default function MovieDetail() {
  const movie = useLoaderData();
  return <h1>{movie.title}</h1>;
}
```

**Modern rule**: Prefer `loader` over `useEffect + fetch`.

---

## 7) Handle Mutations with `action`

Use actions for form submissions and side effects.

```js
{
  path: "login",
  element: <Login />,
  action: async ({ request }) => {
    const formData = await request.formData();
    return authenticate(formData);
  },
}
```

```jsx
import { Form } from "react-router-dom";

export default function Login() {
  return (
    <Form method="post">
      <button type="submit">Login</button>
    </Form>
  );
}
```

No manual `onSubmit` or `fetch` needed.

---

## 8) Navigate

- **Declarative**: `<Link to="/about" />`
    
- **Imperative** (rare): `useNavigate()`
    

```jsx
import { useNavigate } from "react-router-dom";

const navigate = useNavigate();
navigate("/dashboard");
```

---

## 9) Handle Errors per Route

Errors in loaders, actions, or rendering are caught by `errorElement`.

```js
{
  path: "/",
  element: <RootLayout />,
  errorElement: <NotFound />,
}
```

Use `useRouteError()` inside the error component if needed.

---

## 10) Code Split at the Router Level

Lazy-load routes directly in the router.

```js
{
  path: "about",
  lazy: async () => {
    const module = await import("./pages/About");
    return { Component: module.default };
  },
}
```

Cleaner than wrapping routes with `Suspense`.

---

## Modern Best Practices (Checklist)

- Use **`createBrowserRouter`**, not `<BrowserRouter>`
    
- Put data fetching in **loaders**
    
- Use **actions + `<Form>`** for mutations
    
- Treat the URL as **app state**
    
- Co-locate routes, data, and errors
    

---

## One-Sentence Mental Model

> **Modern React Router turns URLs into a declarative state machine for UI, data, and mutations.**