---
tags: 
 - react
 - route
 - term
---

Dynamic routing in **React** (using **React Router v6+**) lets you create routes that change **based on data or user interaction**, such as viewing a specific user, product, or post. Its a great way to [[Storing state in the URL|storing state in the URL]].

Here’s a complete breakdown 👇

---

## 🧠 What is Dynamic Routing?

Dynamic routing means your **URL paths contain variables** (called _route params_) that can change depending on what you’re showing. For example:

```
/users/alice
/users/bob
/posts/123
```

All can be handled by a **single route definition** using a placeholder (e.g. `:userId`, `:postId`).

---

## ☝️ How to use?

Here’s a **simple 3-step guide** to using **dynamic routes** in React Router (v6+).

#### 🧩 1. **Define the dynamic route**

In your router setup (like `App.jsx` or `Routes.jsx`):

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import City from "./pages/City";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="city/:id" element={<City />} />
      </Routes>
    </BrowserRouter>
  );
}
```

✅ The `:id` part is a **route parameter** — it makes the URL dynamic, e.g.  
`/city/1`, `/city/london`, `/city/abc123`

#### 🧩 2. **Create the link with a dynamic value**

Use `<Link>` or `<NavLink>` to navigate with an actual parameter value:

```ad-tip
You can add [[URL#Parameters|query string]] here also. And [[Manipulating Query String|here]] is how to use it.
```

```jsx
import { Link } from "react-router-dom";

<Link to={`/city/${city.id}?lat=${lat}&lng=${lng}`}>{city.name}</Link>
```

```ad-note
Imagine we are passing the data as a state to the globally accessable URL params.
```

✅ When clicked, this goes to `/city/1` (for example).

#### 🧩 3. **Access the dynamic value**

In your target component (`City.jsx`):

```jsx
import { useParams } from "react-router-dom";

function City() {
  // Use params return an object with the property name we use in the route
  const { id } = useParams(); // ✅ read URL parameter
  const [ searchParams, setSearchParams ] = useSearchParams(); // ✅ read query string
  const lat = searchParams.get("lat");
  const lng = searchParams.get("lng");
  return <h2>City ID: {id}</h2>;
}

export default City;
```

✅ `useParams()` gives you an object like `{ id: "1" }`.

#### 🧠 TL;DR

|Step|What you do|Example|
|---|---|---|
|1|Define route with `:param`|`<Route path="city/:id" />`|
|2|Link to it dynamically|`<Link to={`/city/${city.id}`} />`|
|3|Read it in component|`const { id } = useParams();`|

---

## ⚙️ Basic Example

```jsx
import { BrowserRouter, Routes, Route, useParams, Link } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        {/* Dynamic route with :id */}
        <Route path="/posts/:postId" element={<PostDetail />} />
      </Routes>
    </BrowserRouter>
  );
}

function Home() {
  const posts = [
    { id: 1, title: "Hello World" },
    { id: 2, title: "React Rocks" },
  ];

  return (
    <div>
      <h2>All Posts</h2>
      {posts.map((p) => (
        <Link key={p.id} to={`/posts/${p.id}`}>
          <p>{p.title}</p>
        </Link>
      ))}
    </div>
  );
}

function PostDetail() {
  const { postId } = useParams(); // ← Read dynamic part of the URL
  return <h3>Currently viewing post #{postId}</h3>;
}
```

🧩 Here’s what happens:

1. The `path="/posts/:postId"` route declares a **placeholder** for dynamic data.
    
2. When you navigate to `/posts/1`, `useParams()` returns `{ postId: "1" }`.
    
3. You can use that param to fetch or render the right data.
    

---

## 🚀 Fetching Data with Dynamic Route

You can use the `postId` from `useParams` to load data dynamically:

```jsx
function PostDetail() {
  const { postId } = useParams();
  const [post, setPost] = React.useState(null);

  React.useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`)
      .then((res) => res.json())
      .then(setPost);
  }, [postId]);

  if (!post) return <p>Loading...</p>;
  return (
    <article>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </article>
  );
}
```

When you use `useParams`, you’re **reading parts of the URL path** as dynamic variables.  
These values act like _state_, but are stored in the browser’s address bar — meaning:

- They persist across refreshes.
    
- They can be shared via links.
    
- They can be used to fetch or render data dynamically.

---

## 🧭 Nested Dynamic Routes

Dynamic routes can also **nest** for hierarchical data:

```jsx
<Route path="/users/:userId/posts/:postId" element={<UserPost />} />
```

You can access both:

```jsx
const { userId, postId } = useParams();
```

---

## 🧩 When to Use Dynamic Routing

✅ Use dynamic routing when:

- The data or content changes by ID (like `/product/:id` or `/user/:username`)
    
- You want URLs that can be shared/bookmarked
    
- You’re fetching data specific to a route parameter
    

❌ Don’t use dynamic routing when:

- The change is purely UI state (like toggling a modal or tab).  
    → Use `useState` or `useSearchParams` instead.
    

---

## 🧭 Summary

|Concept|Description|
|---|---|
|`:paramName`|Declares a dynamic segment in a route path|
|`useParams()`|Access the current URL parameters|
|`Link`|Navigate dynamically by building URLs with params|
|`useEffect`|Often used to fetch data when a param changes|